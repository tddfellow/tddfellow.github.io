---
layout: post
title: "Build Your Own Testing Framework"
date: 2016-07-15 08:00:06 +0200
comments: true
categories:
- TDD
- testing
- xUnit
- build-testing-framework
- javascript
---

Today we are going to test-drive the testing framework without any external testing framework.
This will be done through test-driving a simple kata (FizzBuzzKata). For example:

- every time we expect test to fail and it doesn't, this is a failing test for our testing framework, that we will be fixing,
- every time we expect test to pass and it doesn't, this is another failing test for our testing framework, that we will be fixing.

For practical reasons, today we are going to use concrete programming language instead of pseudo-code - javascript. Except for small details, that we will point out, the techniques shown here are language-agnostic.

This article is only first one of the series "Build Your Own Testing Framework" == so make sure to stick around for next parts!

Shall we begin?

## FizzBuzzKata

Given the number,

- return `Fizz` when number is divisible by 3,
- return `Buzz` when number is divisible by 5,
- return `FizzBuzz` when number is divisible by 3 and 5,
- return string representation of number otherwise.

## Writing your first test

How do we write our first test, when we don't have a testing framework and we want to create one? - It seems, that we have to design how the test should like in our brand new testing framework.

I personally, would go with xUnit-like design, since it is relatively simple. Given this, we might write our first test and it will look something like that:

```javascript
// test/FizzBuzzKataTest.js

function FizzBuzzKataTest() {
    this.testNormalNumberIsReturned = function() {
        this.assertTrue("1" == fizzBuzz(1));
    };
}
```

This test should fail, because function `fizzBuzz` is not defined, but it doesn't fail, since function `testNormalNumberIsReturned` is never called. In fact, object with `FizzBuzzKataTest` is never being created.

Easiest way to solve that:

```javascript
// test/FizzBuzzKataTest.js

function FizzBuzzKataTest() { ... }

var test = new FizzBuzzKataTest();
test.testNormalNumberIsReturned();
```

If we run this code with `node`:

```bash
node test/FizzBuzzKataTest.js
```

We will get the expected error:

```
/path/to/project/test/FizzBuzzKataTest.js:3
        this.assertTrue("1" == fizzBuzz(1));
                               ^

ReferenceError: fizzBuzz is not defined
```

So, let's define this function:

```javascript
// test/FizzBuzzKataTest.js

function FizzBuzzKataTest() { ... }

function fizzBuzz() {}
```

If we run our test again, we will get the following error:

```
/path/to/project/test/FizzBuzzKataTest.js:3
        this.assertTrue("1" == fizzBuzz(1));
             ^

TypeError: this.assertTrue is not a function
```

Clearly, to fix it we need to define `assertTrue` on `FizzBuzzKataTest` object. Obviously, we do not want our user to define all their assertion for every test suite. This means, that we want to define it on `FizzBuzzKataTest` object outside of the definition of `FizzBuzzKataTest`.

There are two ways to go about it:

- inheritance: make `FizzBuzzKataTest` inherit from some other object function `assertTrue`, or
- composition: make `FizzBuzzKataTest` accept special object with function `assertTrue` defined on it.

I would like to go with composition method, since it gives us more flexibility in the long run:

```javascript
function FizzBuzzKataTest(t) { ... }
```

and the usage of `assertTrue` has to change appropriately:

```javascript
    t.assertTrue("1" == fizzBuzz(1));
```

and `t` has to be created and passed in correctly:

```javascript
function FizzBuzzKataTest(t) { ... }

function fizzBuzz(number) {}

var assertions = {
    assertTrue: function(condition) {}
};

var test = new FizzBuzzKataTest(assertions);
```

If we run the test suite again, we will not get any failure anymore. But we were expecting `assertTrue` to fail, so let's make it fail:

```javascript
    assertTrue: function(condition) {
        throw new Error("Expected to be true, but was false");
    }
```

When we run the test suite, we get:

```
/path/to/project/test/FizzBuzzKataTest.js:11
        throw new Error("Expected to be true, but got false");
        ^

Error: Expected to be true, but got false
```

Now, let's customize the error message a bit:

```javascript
    this.testNormalNumberIsReturned = function() {
        t.assertTrue("1" == fizzBuzz(1), "Expected to equal 1, but got: " + fizzBuzz(1));
    }
```
