---
layout: post
title: Calling and Executing C++ Code from Node.js
---

There is three general ways of integrating C++ code with a Node.js application - although there are lots of variations within each category like: Automation, Shared Library and Addon. Each of them has advantage and disadvantage as well

It's not that hard to execute `C++` code with JavaScript. NodeJS can not only load and/or execute the JavaScript libraries and/or files, but also be extended with native modules (compiled C/C++ code). While this does not mean that you should wipe out your existing JavaScript modules into C++, this knowledge might come in handy in specific use cases.

**Prerequisite**

You will also need below tools installed on your machine, if you are using macOS those are preinstalled :

* make
* g++
* python2.7  

We are ready to compile our first `C++` code into native modules. We will write a simple c++ code that will check whether given number is a prime number and return boolean (true or false) value. So Create a `.cpp` file (I named it prime.cpp)

```
// prime.cpp
#include <nan.h>

NAN_METHOD(IsPrime) {
    if (!info[0]->IsNumber()) {
        Nan::ThrowTypeError("argument must be a number!");
        return;
    }

    int number = (int) info[0]->NumberValue();

    if (number < 2) {
        info.GetReturnValue().Set(Nan::False());
        return;
    }

    for (int i = 2; i < number; i++) {
        if (number % i == 0) {
            info.GetReturnValue().Set(Nan::False());
            return;
        }
    }

    info.GetReturnValue().Set(Nan::True());
}

NAN_MODULE_INIT(Initialize) {
    NAN_EXPORT(target, IsPrime);
}

NODE_MODULE(addon, Initialize);

```

Now install following node_moduels `nan` `node-gyp` and update your package.json like this :

```

{
  "name": "calling-cpp-into-nodejs",
  "version": "1.0.0",
  "dependencies": {
    "nan": "^2.6.1",
    "node-gyp": "^3.6.0"
  },
  "scripts": {
    "compile": "node-gyp rebuild",
    "start": "node test.js"
  },
  "gypfile": true
}

```

Here in the "scripts" section:


* `"compile": "node-gyp rebuild"` — for C++ code compilation
* `"start": "node main.js"` — for our main executable script
```

Now create a bindings.gyp fie (It's a Configuration file for `node-gyp`) and add your `prime.cpp` file  in the array under sources:

```
{
  "targets": [
    {
      "include_dirs": [
        "<!(node -e \"require('nan')\")"
      ],
      "target_name": "addon",
      "sources": [ "prime.cpp" ]
    }
  ]
}
```

Yeah! We are done. Now it's time to compile your C++ file run `npm run compile` it will start node-gyp for compiling our C++ files from bindings.gyp sources and output should be like (if everything is okay):

```
node-native-addons-example@1.0.0 compile /Users/marcin/projects/node-native-addons-example
node-gyp rebuild
CXX(target) Release/obj.target/addon/main.o
SOLINK_MODULE(target) Release/addon.node
```

Yepiee! you are ready to use your first compiled native module. For testing we are using a .js file named test.js and its look like below :

```
const {IsPrime} = require('./build/Release/addon'); // native c++

const number = 654188429; // thirty-fifth million first prime number (see https://primes.utm.edu/lists/small/millions/)


console.time('start ....');
console.log(`checking whether ${number} is prime... ${IsPrime(number)}`);
console.timeEnd('finished ...');

```

We are done! Happy Coding :)