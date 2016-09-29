---
layout: post
title: "Meet Duck Type"
description: "Have you ever heard of the Duck Type and Duck Typing? Did you know that you need to contract-test your duck types? Learn how!"
date: 2016-09-18 16:37:33 +0200
comments: true
categories:
- javascript
- testing
- duck-typing
---

## Duck type

Duck type is the concept in the domain of the type safety that represents objects, that pass a so-called "Duck Test":

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.

In terms of programming language, it might look like this:

```javascript
function Duck() {
    this.swim = function (coordinates) { ... };
    this.quack = function (sentence) { ... };
}

function RoboDuck() {
    this.swim = function (coordinates) { ... };
    this.quack = function (sentence) { ... };
}

// .. and so on ..
```

<!--more-->

The point is that the public interface has methods `swim()` and `quack()`. This is how you identify the duck in a programming language. This concept is very similar to the concept of the `interface` in programming languages that have one, but it is not enforced in any way by the programming language.

Duck typing is mostly natural in dynamic languages, where it is possible to send any message to any object and the check if that is something possible will happen at runtime. In static languages, it is still possible to use duck typing via some sort of Reflection.

### Contract test for duck types

In a dynamic language, it is important to make it obvious, that something is implementing certain duck type by writing one test suite for all implementers and executing it against them. For example:

```javascript
[Duck, RoboDuck].forEach(function (duckType) {
    runTestSuite(function (t) {
        this.testItSwimsLikeADuck = function () {
            var duck = new duckType();
            duck.swim({x: 5, y: 7});
            t.assertThat(duck).swamTo({x: 5, y: 7});
        };

        this.testItQuacksLikeADuck = function () {
            var duck = new duckType();
            duck.quack("hello world");
            t.assertThat(duck).quacked("hello world");
        };
    });
});
```

This test suite has to go only through the Duck type public interface. If it is not possible to test behavior through it, one should test at least the function signatures, for example:

```javascript
this.testItSwimsLikeADuck = function () {
    var duck = new duckType();
    var swim = duck.swim;
    t.assertEqual("function", typeof(swim));
    t.assertEqual(1, swim.length);
};
```

Not doing contract tests for your ducks may result in a passing test suite and broken production code. For example, when one duck and its test suite have been updated, but others haven't.

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
