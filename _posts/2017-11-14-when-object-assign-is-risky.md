---
layout: post
title: When Object.assign() Is Risky
---

Javascript `Object.assign` allows you to copy one set of an object non-inherited properties to another.It has a signature of Object.assign(target, ...sources). The target object is the first parameter and is also used as the return value. Object.assign() is useful for merging objects or cloning them shallowly. code sample below :

```
const adminPrivileges = {
  canCreate: true,
  canDelete: true,
  canUpdate : true,
  canRead : true
};

const userPrivileges = Object.assign({}, adminPrivileges, {
  canCreate: false,
  canDelete: false,
  canUpdate : false,
  canRead : true
});

console.log(userPrivileges);
//  { canCreate: false,
//   canDelete: false,
//   canCreate: false,
//   canRead: false }

```

`But` The *Risky*   is that if you forget to include {} will result in accidentally modifying the original object causing nasty consequences later and its hard to notice.

```
const userPrivileges = Object.assign(adminPrivileges, {
  canCreate: false,
  canDelete: false,
  canUpdate : false,
  canRead : true
});

console.log(userPrivileges);
//  { canCreate: false,
//   canDelete: false,
//   canCreate: false,
//   canRead: true }

console.log(adminPrivileges);
//  { canCreate: false,
//   canDelete: false,
//   canCreate: false,
//   canRead: true }

```
Ahhh! Original source is modified here. So Instead of `Object.assign()` we can use `Spread` properties. like :

```
const userPrivileges = {
  ...adminPrivileges,
  canCreate: false,
  canDelete: false,
  canUpdate : false
};
console.log(userPrivileges);
//  { canCreate: false,
//   canDelete: false,
//   canCreate: false,
//   canRead: true }


console.log(adminPrivileges);
//  { canCreate: true,
//   canDelete: true,
//   canCreate: true,
//   canRead: true }

```


And Two objects can be directly merged if the overriding properties also come from a previous object too:
```
const userPrivileges = {
  canCreate: true,
  canDelete: true
};
const anotherTypePrivileges = { ...adminPrivileges, ...userPrivileges };
```

 Happy coding, be careful ^-^