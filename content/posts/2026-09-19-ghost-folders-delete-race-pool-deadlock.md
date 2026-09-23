---
title: "Ghost folders, a delete race, and a pool deadlock"
date: 2026-09-19
slug: kb-file-delete-race-and-pool-deadlock
images: [images/og/kb-file-delete-race-and-pool-deadlock.png]
tags: [concurrency, postgres, nodejs, filesystem, multi-tenant]
description: "Ghost directories, a delete-versus-extraction race, and a connection-pool deadlock in a multi-tenant file store: three concurrency lessons from one teammate's fix."
---

A teammate on our platform team shipped a fix I read closely, because it packs three concurrency lessons into one day of commits. The feature is a knowledge base: users upload files into folders, we extract text from each file, and we index that text into a vector store for search. It worked, but three things were wrong once real traffic hit it: deleting a file left empty folders behind, a delete racing an extraction could resurrect a folder with no database row, and a bulk upload could deadlock the database connection pool.

This post is what broke, why, what changed, and how it was checked without a full-scale bulk test on a real cluster. Four commits, one day. Identifiers are generic.

## The setup

Each file lives on disk under `{tenant}/{kb}/{subdir}/{name}`. Uploading into `reports/2025/` does an implicit `mkdir -p`. Directories have no database row. The browse endpoint lists folders straight off the filesystem, so an empty folder on disk is a folder in the UI.

Extraction runs in a separate worker. When it finishes, it calls back into the control plane. The callback writes the extracted markdown next to the original file (`report.pdf` → `report_extracted.md`, same directory) and dispatches an indexing job. Writes on the same KB are serialized with a Postgres advisory lock (`pg_advisory_lock`). It is session-scoped, so it is pinned to one pooled connection for the lock and unlock.

## Root cause 1: delete only unlinked the file

Single-file delete called `unlink` and stopped. The parent folders it emptied stayed on disk, so after deleting the last file in `reports/2025/`, both `reports/2025/` and `reports/` kept showing up in browse. Upload created folders implicitly; delete never removed them. The lifecycle was asymmetric.

## Root cause 2: the extracted sibling blocked the prune, and a delete could race the extraction

The first attempt at a prune walk broke on the extracted `.md`. The extracted file sits in the *same* directory as the original. Pruning the original's parents first meant the lingering sibling made the first `rmdir` return `ENOTEMPTY`, the walk stopped at the deepest level, and the ghost folders survived anyway.

The worse version was a race. A delete could prune the directory and remove the row *after* the extraction callback read the file but *before* it saved the markdown. That save does `mkdir -p`. So the late write rebuilt a directory that had just been pruned and dropped an orphaned `.md` with no database row pointing at it.

## Root cause 3: concurrent lock entrants deadlocked the pool

This one only shows up under a bulk upload. Every extraction callback takes the per-KB advisory lock, and the lock is pinned to a pooled connection. When N callbacks for the same KB arrive at once and N is at or above the pool size, each one grabs a connection and blocks inside `pg_advisory_lock` waiting for the holder. The holder then needs a connection of its own to run its queries — and there are none left. Everyone waits on everyone. The pool is dead until something times out.

{{< mermaid >}}
flowchart TB
  subgraph before["Before: each waiter pins a connection"]
    C1["callback A"] --> K1["conn 1 · blocked on lock"]
    C2["callback B"] --> K2["conn 2 · blocked on lock"]
    H1["lock holder"] --> N1["needs a conn to query"] --> D1["pool empty → deadlock"]
  end
  subgraph after["After: waiters queue in-process, holding nothing"]
    Q1["callback A"] --> G["in-process FIFO gate (no conn)"]
    Q2["callback B"] --> G
    G --> HEAD["queue head only"] --> L2["takes advisory lock + a conn"]
  end
{{< /mermaid >}}

## The fix

**Prune empty parents, stop at the root, never escape it.** A delete-and-prune call in the storage adapter. It walks up from the file's directory calling `rmdir`, and the loop guard keeps it strictly below the KB root, so a wrong KB id prunes nothing instead of walking into another tenant.

```ts
let dir = dirname(fullPath);
while (dir.startsWith(root + "/")) {
  try {
    await rmdir(dir);
  } catch (err) {
    const code = (err as NodeJS.ErrnoException).code;
    // The unlink already succeeded. A parent we cannot remove must only
    // STOP the walk — it must never fail a delete that is already done.
    if (code !== "ENOTEMPTY" && code !== "EEXIST" && code !== "ENOENT") {
      log.warn({ action: "prune-stopped", dir, code });
    }
    break;
  }
  dir = dirname(dir);
}
```

`ENOTEMPTY`/`EEXIST` means a sibling is present, including a hidden one — an NFS `.nfsXXXX` silly-rename of an open file, or a stray dot-file that the browse listing filters out. `EPERM`/`EBUSY` is a transient lock. In every case the walk stops and logs the surprising codes. It does not re-throw, because the file is already gone and the delete has to report success.

**Delete the sibling first, then prune.** The service removes the extracted `.md` before pruning the original's parents, so the first `rmdir` sees an empty directory.

**Hold the lock across read and remove.** The delete re-reads and removes under the per-KB lock. The extraction callback also re-reads inside the lock and only acts if the file is still in `extracting`. If the row is gone or the status moved on — a delete won, a cancel landed, or the queue redelivered the callback — it drops the write instead of resurrecting the folder.

**Move the slow, lockless work out of the critical section.** Two calls did not need the lock and were making it worse. Vector cleanup is a network call to the embedding service; provider resolution and the indexing dispatch are their own round-trips. All of them moved outside the lock. Now the lock spans only the storage prune and the row delete. A slow embedding service no longer stalls every operation queued on that KB.

```ts
// Vectors first, outside the lock — best-effort, a failure is logged not thrown.
await this.deleteVectors(collection, fileId);

await this.withKeyLock(kbId, async () => {
  const file = await this.files.findByIdOrFail(kbId, fileId);
  if (file.extractedPath) {
    await this.storage.deleteFile(tenantId, file.extractedPath); // sibling first
  }
  await this.storage.deleteFileAndPruneEmptyParents(tenantId, file.path, kbId);
  await this.files.delete(fileId);
});
```

**Put an in-process gate in front of the advisory lock.** This is what fixes the deadlock. The repository is a singleton, so it keeps one promise chain per KB. Entrants queue on that chain holding *no* database connection. Only the head of the queue goes on to take the advisory lock and a connection. The Postgres lock still serializes across replicas; the gate only serializes within the process, which is exactly the part that was starving the pool.

```ts
async withKeyLock<T>(key: string, fn: () => Promise<T>): Promise<T> {
  const prev = this.gates.get(key) ?? Promise.resolve();
  let release!: () => void;
  const gate = new Promise<void>((r) => (release = r));
  this.gates.set(key, gate);
  await prev; // wait our turn, holding nothing
  try {
    return await this.withDbAdvisoryLock(key, fn);
  } finally {
    release();
    if (this.gates.get(key) === gate) this.gates.delete(key);
  }
}
```

## How it was verified without the full-scale test

The real repro is a bulk upload with concurrent extraction callbacks racing concurrent deletes on a live cluster. Nobody wanted to depend on that, so each failure mode was pushed down to something smaller. This part is what I would copy into my own work.

- **Filesystem adapter, real disk.** The prune runs against a real temp directory with only `rmdir` mocked, so an errno can be injected without faking the filesystem. Cases: prune the whole empty chain but never the KB root; stop at a sibling file; stop at a sibling directory; tolerate `ENOENT` on a double delete; leave a folder alone when a hidden `.nfs01234` sibling keeps it non-empty; a wrong KB id prunes nothing and escapes nothing; `EPERM` logs and stops without throwing; and reject tenant-prefix and path-traversal inputs.
- **Service, call order and lock use.** Unit tests assert the order — vectors, extracted sibling, prune, row — and that the work runs under `withKeyLock`. A regression that silently drops the lock fails a test instead of production. More tests drop the extraction callback when the re-read shows the file deleted or no longer `extracting`, and confirm no write and no dispatch happen.
- **Integration, the worker's real path.** Upload a file into `foo/bar`, PUT the extracted content through the internal endpoint the worker uses, delete the file, then browse and assert no folders survive — including the case where extraction already ran and left a sibling.
- **The deadlock, reasoned and shrunk.** You do not need a bulk load. Set the pool to N and fire N+1 concurrent lock acquisitions on one KB; the old path hangs, the gated path drains one at a time. The structural argument is simpler: waiters now hold no connection, so the holder can always get one.

## What could be better

- Directories still have no identity. No description, no owner, no rename without rewriting every child path. That was a deliberate call, but it is the reason browse leans on storage being healthy.
- Storage is the source of truth for the folder list. If a folder is removed out-of-band while rows still point under it, those files vanish from browse but still stream by id.
- Single-file vector cleanup is best-effort now. If the embedding service is down, the file and row go but the vectors linger, and there is no sweep to reclaim them yet.
- The advisory lock hashes the KB id, so two different KBs can share a lock. That only costs some serialization, not correctness, but it is a sharp edge.
- Hidden `.nfs` siblings still leave a folder behind. Stopping beats fighting open handles.

## Takeaways

- Make lifecycles symmetric. If upload creates folders, delete should remove them — the missing half is where ghosts live.
- Never let cleanup fail the thing it is cleaning up after. Once the unlink succeeds, an un-removable parent is a reason to stop, not to error.
- Keep slow and networked work out of a lock. Vectors, provider lookups, and dispatch belong outside the critical section.
- A session-scoped database lock pins a connection. Queue in-process first, holding nothing, so waiters cannot starve the pool.
- Push each failure mode to the smallest test that still exercises it — real disk with one mocked call, asserted call order, the worker's real endpoint — instead of leaning on a full-scale run.
