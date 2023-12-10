---
layout: post
title: One-Way vs. Two-Way Data Binding
---

Two-way binding is simply a way to sync data to a view and vice versa  (mvvm pattern). One-way data binding refers to the design pattern found in component based frameworks, where a child component cannot modify data passed by a parent component, as that creates spaghetti code.


![One-way vs Two-way data binding](https://image.slidesharecdn.com/qew2fe1qfouq3ez0irrw-signature-a5e16f8a5f9bf5c06bfd992151090953475efa37da75f9eed6674474188d6d90-poli-141218052940-conversion-gate02/95/angular-js-10-638.jpg?cb=1418892818)
> Image source slideshare

#### One-Way Data Binding

* One-way data binding refers to the design pattern found in component based frameworks, where a child component cannot modify data passed by a parent component, as that creates spaghetti code.

* One way binding only binds the value of the model to the view and does not have an additional watcher to determine if the value in the view has been changed by the user.

#### Two-Way Data Binding

* Two-way data binding provides the ability to take the value of a property and display it on the view while also having an input to automatically update the value in the model. You could, for example, show the property "name" on the view and bind the text input value to that same property. So, if the user updates the value of the text input the view will automatically update and the value of this parameter will also be updated in the model.

* Two-way binding is simply a way to sync data to a view and vice versa (mvvm pattern).