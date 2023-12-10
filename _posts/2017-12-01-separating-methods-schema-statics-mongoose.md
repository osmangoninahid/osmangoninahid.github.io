---
layout: post
title: Separating Schema, Method and Statics in Mongoose NodeJS
---

![mongoose-nodejs-mongodb](https://raw.githubusercontent.com/daton89-topperblues/mongoose-transactions/master/docs/img/mongoose-transactions.png)

Database logic should be encapsulated within the data model. Mongoose provides 2 ways of doing this, methods and statics. Methods adds an instance method to documents whereas Statics adds static "class" methods to the Models itself. like example below:

```
const bookSchema = mongoose.Schema({
  title: {
    type : String,
    required : [true, 'Book name required']
  },
  publisher : {
    type : String,
    required : [true, 'Publisher name required']
  },
  thumbnail : {
    type : String
  }
  type : {
    type : String
  },
  hasAward : {
    type : Boolean
  }
});

//method
bookSchema.methods.findByType = function (callback) {
  return this.model('Book').find({ type: this.type }, callback);
};

// statics
bookSchema.statics.findBooksWithAward = function (callback) {
  Book.find({ hasAward: true }, callback);
};

const Book = mongoose.model('Book', bookSchema);
export default Book;
```

It's awesome right? Yeah!. But what if, you have a complex schema , large amount of field and schemas the number of methods and statics are pretty large.

There's a good chance that you're putting too much stuff into the model logic rather than making use of modules that hold functionality that you can then wire together in your model.

You wrote them into schema file like above, it's fine. Now see your schema schema file that are too big. Obviously it's not looking good now, right? Yes.

There is a great way to get reuse as well as to separate all schema, statics and methods by the mongoosy way to do this is actually to make use of their plugins architecture. Make another sub dir named `plugins` and put all methods and statics file there. like below example:

```
// ./plugins/find-by-type.js

function findByType(type, callback) {
  this.find({ type: new RegExp(type, 'i') }, callback);
}

function findByTypePlugin(schema, options) {
  schema.statics.findByType = findByType;
}

module.exports = findByTypePlugin;

```

Then in your Schema file you can just require and add them

```
const findByTypePlugin = require('./plugins/find-by-type');
const bookSchema = mongoose.Schema({
  title: {
    type : String,
    required : [true, 'Book name required']
  },
  publisher : {
    type : String,
    required : [true, 'Publisher name required']
  },
  thumbnail : {
    type : String
  }
  type : {
    type : String
  },
  hasAward : {
    type : Boolean
  }
});

bookSchema.plugin(findByTypePlugin);
//add more like this

const Book = mongoose.model('Book', bookSchema);
export default Book;

```

Happy Coding :)