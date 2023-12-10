---
layout: post
title: Use Cases for Using Let and Const in JavaScript
---

There are now two new ways to declare variables in JavaScript: `let` and `const`.The only way to declare a variable in JavaScript was to use the keyword `var`. To understand why let and const were added, it’s probably best to look at an example of when using var can get us into trouble.

```
function getThing(isDoes) {
  if (isCold) {
    var foo = 'Grab a jacket!';
  } else {
    var bar = 'It’s a shorts kind of day.';
    console.log(foo);
  }
}
```

### let and const
Variables declared with `let` and `const` eliminate this specific issue of hoisting because they’re scoped to the block, not to the function. Previously, when you used var, variables were either scoped globally or locally to an entire function scope.

If a variable is declared using `let` or `const` inside a block of code (denoted by curly braces `{ }`), then the variable is stuck in what is known as the temporal dead zone until the variable’s declaration is processed. This behavior prevents variables from being accessed only until after they’ve been declared.

### Rules for using let and const
`let` and `const` also have some other interesting properties.

* Variables declared with `let` can be reassigned, but can’t be redeclared in the same scope.
* Variables declared with `const` must be assigned an initial value, but can’t be redeclared in the same scope, and can’t be reassigned.

### Use cases
The big question is when should you use let and const? The general rule of thumb is as follows:

* Use `let` when you plan to reassign new values to a variable, and
* Use `const` when you don’t plan on reassigning new values to a variable.
Since `const` is the strictest way to declare a variable, we suggest that you always declare variables with const because it'll make your code easier to reason about since you know the identifiers won't change throughout the lifetime of your program. If you find that you need to update a variable or change it, then go back and switch it from `const` to `let`.


Happy coding :)