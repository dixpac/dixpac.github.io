---
title: "How to setup ES6 in your Rails application"
layout: post
date: 2016-09-20 13:00
tag:
- rails
- JavaScript
blog: true
---

**ECMAScript6(ES6)** is the new version of JavaScript language, with a lot
of new and shinny features. Lets take a look at some of them.

### Classes

For some a long waited feature in Js, for others worst nightmare,
classes. Similar as in your other favourite languages as ruby, java,
etc... you  can now create classes in Js, like this:

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  get name() {
    return this.firstName + " " + this.lastName;
  }

  set name(name) {
    var names = name.split(" ");

    this.firstName = names[0];
    this.lastName = names[1];
  }
}

var blake = new Person("Darth", "Vader");
blake.name = "Light Vader"
console.log(blake.name); // => Light Vader
```

### Interpolation

ES6 finally deliver more powerful string syntax in JavaScript. One of those
powerful new features is *String interpolation* which is awkwardly similar to Ruby
String interpolation....well stealing from the best is always good thing to do :)  
To use interpolation in ES6 you surround string in a backticks (``) instead of
regular quotes ("")

ES5 way:  

```js
var name = 'Vader'
console.log("Your name is: " + name)
```

ES6 way:  

```js
var name = 'Vader'
console.log(`Your name is: ${name}`)
```

### Fat Arrows

Another appealing feature in ES6: fat arrows. Fat arrows allow us to bind a function to the current value of this. First, let’s take a look at how we can handle this without a fat arrow.

With ES5 we have to keep a reference to the current value of this when defining the function:

```js
var self = this;

$("button").on("click", function() {
  // do something with self
});
```

In ES6 you could simply use fat arrows to achive same thing

```js
$("button").on("click", () => {
  // do something with this
});
```

### Default arguments

You can also set default values to function parameters now, just as you would expect

ES5 way:

```js
"use strict";

var hello = function hello() {
  var name = arguments.length <= 0 || arguments[0] === undefined ? "Vader" : arguments[0];

  alert(name);
};
```

ES6 way:

```js
var hello = function(name = "Vader") {
  alert(name);
}
```

### Splats

Splats, allow you to collect additional arguments passed to your function as an array.
ES6 refers to them as **rest arguments**

ES5 way:

```js
"use strict";

var ingredients = function awards(first, second) {
  var salt = first;
  var peper = second;

  for (var _len = arguments.length, others = Array(_len > 2 ? _len - 2 : 0), _key = 2; _key < _len; _key++) {
    others[_key - 2] = arguments[_key];
  }

  var love = others;
};
```

ES6 way:

```js
var ingredients = function(first, second, ...others) {
  var salt = first;
  var peper = second;
  var love = others;
}
```

**These are just some of hounorable mentions and by all means not the only ones, here is
more detailed [list](http://es6-features.org/#Constants).**

### Current state and browsers

Browsers are implementing new features in a fast pace, but it would still take some time until we could actually use ES6. Maybe years. Fortunately, we have [Babel.js](https://babeljs.io/).
We can use all these new features today without worrying with browser compatibility.

**Babel.js is just a pre-processor. You write code that uses these new features, which will be exported as code that 
browsers can understand, even those that don’t fully understand ES6.**

### Setup Rails

Finally, to use ES6 in Rails now I will add gem [sprockets-es6](https://github.com/TannerRogalsky/sprockets-es6).
`sprockets-es6` is Sprockets transformer that converts ES6 code into vanilla ES5 with Babel.js,
so I need to add `babel-transpiler` gem also.

Gemfile

```ruby
gem 'sprockets', '>= 3.0.0'
gem 'sprockets-es6'
gem 'babel-transpiler'
```

**Note** for this to work you must use at least sprockets 3.
ES6 will ship by default with sprockets 4 in the future and hence by default with
Rails, but until then you need to add ES6 support manually.

After adding required gems, we now have ability to write ES6 in our Rails app,
add following file and try it!

 `application/assets/javascripts/hello.js.es6`

```js
var fn = (name) => {
	console.log(`Hello %{name}`);
}

fn('Vader');
```

**Note** you must add *es6* extension to you *js* files

Start rails app an open browsver console and you should see console message!

R'n'R
