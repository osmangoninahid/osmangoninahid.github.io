---
layout: post
title: Mongo DB change stream new Feature of Mongo 3.6
---

Those days are gone when people didn't expect information for momentary access to data. Nowadays expectation everywhere is real-time action and result. Something is happened? not 1 day ago, not even 1 min ago but just now. Being notified constantly on any change of data points a bit critical from many application nowadays. So the upcoming `MongoDB3.6` introduces a new `$changeStream` aggregation pipeline operator.

### Change Stream:
Change streams are being implemented in the driver with a new aggregation operator, `$changeStream` and watch method in the API. Where able to specify a `$change` stage at the beginning of our pipeline and request that notifications are sent for specific changes or hooking something to a particular collection.

### Change Streams in code:
```
var MongoClient = require('mongodb').MongoClient;
MongoClient.connect("mongodb://localhost:27017/myproject?readConcern=majority")
 .then(function(client){
   let db = client.db('myproject')
   // specific table for any change
   let change_streams = db.collection('documents').watch()
      change_streams.on('change', function(change){
        console.log(change);
      });
  });

// with aggregation on whole database

change_streams = client.db.collection.watch([
    {'$match': {
        'operationType': {'$in': ['insert']}
    }},
    {'$match': {
        'level': {'$lt': 350}
    }}
]);

for change inchange_streams:
    do_some_magic();
```
>Note : If you are using node.js, you need to use following node module to get it working:

```
"dependencies": {
    "mongodb": "mongodb/node-mongodb-native#3.0.0"
 }
 ```

### Change Stream action / operation Types:
* Insert
* Delete
* Replace (everything but the unique id)
* Update
* Invalidate (in cases of an invalid cursor being returned)

Thanks for reading, Happy coding :)