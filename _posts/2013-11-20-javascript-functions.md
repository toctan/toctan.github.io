---
layout: post
title: JavaScript Functions
tags: JavaScript js function
---


> Functions are the very best part of JavaScript, it's where the most
> power is, it's where the most beauty is.<br>
> <cite>— [Douglas Crockford][]</cite>


## Function Objects

```js
// function expression
var add = function (a, b) {
    return a + b;
}

// function statement
function add (a, b) {
    return a + b;
}
```

Functions in JavaScript are objects. Either the expression or
statement above produces an instance of a function object, which
inherits from `Function.prototype`. It sounds odd that `function` can
inherit but in JavaScript it can. So, __function in JavaScript have
methods__.

Since functions are objects, they can be used like any other value.  A
function literal can appear anywhere that an expression can appear,
which makes it a [first class citizen][], it may be

- passed as an argument or a parameter
- assigned to a variable
- returned from a function
- stored in an object


## var & return

`var` statement is used to declare and initialize variables within a
function. It will get split into two parts, the declaration part will
get hoisted to the top of the function, initialized with `undefined`,
while the initialization part turns into an ordinary assignment.

```js
var a = 0, b;
// expands to =>

var a = undefined,
    b = undefined;
    ...
a = 0;
```

Every function in JavaScript returns something, if you do not implicitly
return a value, the return value is `undefined`. There is a special case
with constructors, which will return `this`.


## function statement vs function expression

The function produced by function statement also gets hoisted, which makes
it available everywhere in the scope. So it can be called before the actual
definition in the source.

```js
foo(); // => hello world

function foo() {
    console.log('hello world');
}
```

Due to the hoisting caused by the `var` statement, if you try to call a
function produced by the expression form before where it's defined, you
will get an type error.

```js
foo;   // =>undefined
foo(); // => TypeError: undefined is not a function

var foo = function() {};
```

For more information on this, go read
[Named function expression demystified][] by _Juriy Zaytsev_.


## Invocation

The invocation operator `()` is a pair of parentheses which can
contain zero or more expressions. __This is no runtime error when the
number of arguments does not match the number of parameters.__ When
there are too many arguments, the extra ones is ignored. And when
there are too few, the missing ones is filled with `undefined`.

```js
var bar = function (a, b) {
    console.log(b)
}

bar(1, 2, 3, 4) // It's completely OK.
bar(1)          // => undefined
```


## this & arguments

`arguments` and `this` are the two pesudo parameters every function
will receive. When a function is called, in addition to all the
declared parameters, it also gets a special array-like parameter which
contain all the arguments from the invocation. The extra arguments can
be also accessed in `arguments`.

```js
var baz = function () {
    console.log(arguments)
}

baz('hello', 'world')
=> ["hello", "world"]
```

Similar to `self` in many other object oriented programming languages,
`this` contains a reference to the object of invocation. `this` allows
a single function object to service many functions, it's key to prototypal
inheritance.

```js
function Foo() {}
Foo.prototype.method = function() {
    console.log(this);
};

function Bar() {}
Bar.prototype = Foo.prototype;

new Bar().method();
// => Bar {method: function}
```


JavaScript functions can be invoked in four ways, which differ in how
`this` is initialized.

```js
// Function form
// `this` is bound to global object in ES3, undefined in ES5
func(arguments)

// Method form
// `this` is bound to whatever the function is called on
obj.method(arguments)

// Constructor from
// `this` is bound to the new object get greated
new func(arguments)

// Apply form
// `this` is bound to the first parameter
obj.apply(thisObj, [1, 2, 3])
obj.call(thisObj, 1, 2, 3)

// apply accept an array of arguments,
// while call accept arguments individually
```


## Resources

- [Function the Ultimate][] <br> A great talk on JavaScript functions by
  _Douglas Crockford_, highly recommended!!!
- [JavaScript Garden][] <br> All about the quirky parts of JavaScript.


[Douglas Crockford]: http://www.crockford.com/
[first class citizen]: http://en.wikipedia.org/wiki/First-class_citizen
[Named function expression demystified]: http://kangax.github.io/nfe/
[Function the Ultimate]: http://www.youtube.com/watch?v=ya4UHuXNygM
[JavaScript Garden]: http://bonsaiden.github.io/JavaScript-Garden/
