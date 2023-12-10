---
layout: post
title: ES6 Style Module Import and Destructuring Assignment
---

Though I'm a big fan of ES6 and always I try to do everything with ES6 instead of ES5 but sometimes have to write JS code into ES5. That's why I try to make the ES6 flavour into ES5 codes.

In JavaScript it is often more clear to only import parts of modules/objects when they are needed. Before ES6 this was a cumbersome task which required assigning to multiple variables. For example, in Node.js:

```
const mod = require('./PartialModule');
const property_one = mod.property_one;
const property_two = mod.property_two;

property_one();
// Message for propertyOne
property_two();
// Message for propertyTwo

```
Here is the PartialModule.js code :

```
module.exports = {
  property_one: function() {
    console.log('Message for propertyOne');
  },
  property_tow: function() {
    console.log('Message for propertyTwo');
  }
};

```
By ES6 import we can write like this :

```
const { property1, property2 } = require('./partialModuleImportsToRequire');

property_one();
// Message for propertyOne
property_two();
// Message for propertyTwo

```

Although this looks like a special import syntax, it actually is derived directly from two ES6 features; `Destructuring Assignments and Shorthand Property Names` .

### Destructuring Assignments

[Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables.
Destructuring Assignments specify how to insert values from one data structure from another (both objects in this case). To visualize this better :

```
[a, b] = [10, 20];
console.log(a); // 10
console.log(b); // 20
[a, b, ...rest] = [10, 20, 30, 40, 50];
console.log(a); // 10
console.log(b); // 20
console.log(rest); // [30, 40, 50]

({ a, b } = { a: 10, b: 20 });
console.log(a); // 10
console.log(b); // 20


// Stage 3 proposal
({a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40});
console.log(a); // 10
console.log(b); // 20
console.log(rest); //{c: 30, d: 40}
```

We can replaced the import with the object it returns

```
const {
  property_one: property_one,
  property_two: property_two
} = {
  property_one: function() {
    console.log('Message for propertyOne');
  },
  property_two: function() {
    console.log('Message for propertyOne');
  }
}

```

So means, took the left property name `property_two`, find it in the object on the right side, then assign it to a variable named `property_two`. The right side name could be changed to assign `property_two` from the original object to a different property name.

Then, when two properties have the same name, a [Shorthand](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer) can be used so that only one of them has to be entered. Then if both properties are put on one line you get the original syntax, like below:

```
const { property_one, property_two } = ...
```