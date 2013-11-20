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

Actually, the function statement is just a short-hand for a `var`
statement with a function expression.

```js
function foo() {}
// expands to =>

var foo = function foo() {};
// expands to =>

var foo = undefined;
foo = function foo() {};
```

Every function in JavaScript returns something, if you do not implicitly
return a value, the return value is `undefined`. There is a special case
with constructors, which will return `this`.


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

`this` contains a reference to the object of invocation. It's very
similar to `self` in other object oriented programming languages.

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

[Function the Ultimate][]<br> A great talk on JavaScript functions by
Douglas Crockford, highly recommended!!!


[Douglas Crockford]: http://www.crockford.com/
[first class citizen]: http://en.wikipedia.org/wiki/First-class_citizen
[Function the Ultimate]: http://www.youtube.com/watch?v=ya4UHuXNygM
