---
layout: post
title: No more express-generator on API building with NodeJS
---


<img src="https://user-images.githubusercontent.com/6789760/37593802-7b1d071c-2b9d-11e8-9e5d-c0d778fa56dc.jpg" alt="spready" style="width: 600px;"/>

Combined (Front-end and Back-end) application is no more nowadays. The way of doing front-end and backend in same application with same stack is old now. The problem with this model is that it tightly couples front-end and back-end development into a single development stack and imposes restraints on technology choices.

But whilst this has been the best solution for some time, this was primarily due to poor performing web browsers not being able to handle complex operations that are required to run a modern day website.

However, IE7 was a long time ago and browsers have come an extremely long way since then. They have evolved from simple page viewers that were limited to presenting content and basic DOM manipulation, to becoming an extension of our Operating Systems.

Since the emergence of more powerful and high performance web browsers we have the opportunity to not only allow more of the processing power to be handled by the browser, but to define a clear separation of front-end and back-end development and move away from this tightly coupled model, it's called **Split Stack Development Model**.

### Split Stack Development

Split stack development is an architecture pattern that separates front-end and back-end development into two separate `stacks` which function independently and communicate through an API.

By decoupling front-end and back-end development you eliminate any restraints on technology choices that each may have imposed on the other. This results in completely independent stacks used on the client and the server. For example, you could build the back-end using ExpressJS(NodeJS) or Python whilst the front-end could be built using Angular or React, a stack that was difficult to achieve previously.

So, We are writing web applications with **Split Stack Development Model** . If you are going to build an API service with `ExpressJS` you won't go to create all files and file structure from ground if anyone able to do with his/her stamina then fine but it requires few time to make a well structured boilerplate. That's why `ExpressJS` provide us a generator/cli named [express-generator](https://www.npmjs.com/package/express-generator) to get started with express is to utilize the executable express quickly.

Well, You can try this one if you didn't yet and look, We are not making Combined application If we follow the **Split Stack Development Model** that `express-generator` is not good fit for us. Cause we don't need any view engine or library like `ejs, pug, hbs` etc and do not need favicon.ico , serve-favicon etc.

As a noob or new comer developer loosing most of the time to make development environment and structure of a project where should write business logics, controllers and how to arrange files and structure with good naming conventions.

Though express-generator is not only for API building it was aimed combined application building there is a fundamental executable express app not only API focused.

We only need a lean scaffolding to start quickly with minimal and well structured executable express application. It will be plus point if we find any example with database integration and at least one third-party library or module integration example.

Based on those problem and focused only API building there is a generator/cli to Quick start with express is to utilize the executable RESTFul web-service with well structured codebase named **[Spready](https://www.npmjs.com/package/spready)**

## Spready

**Spready** will help you to create a RESTFul backend API service including basic CRUD functionalities with NodeJS, Express.js and MongoDB(mongoose) with only 1 or 2 command. Where basic starter files `bin/www` and `app.js` is like express-generator and each feature and/or module in under `modules` directory with module name, Into a specific module you will find 3 separate directory `controllers` , `models` and `routes`.

* **Controllers** : place for writing business logics and routes callback.
* **Models** : To write database schema or Model.
* **Routes** : Where defines endpoints, middlewares and connecting route with router callback functions for each endpoint. finally all modules routes are exported to starter file (`app.js`).

For more details visit :
[Spready](https://github.com/osmangoninahid/spready)-https://github.com/osmangoninahid/spready