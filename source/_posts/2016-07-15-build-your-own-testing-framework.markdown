---
layout: post
title: "Build Your Own Testing Framework"
date: 2016-07-15 08:00:06 +0200
comments: true
categories:
- TDD
- test-driven-development
- testing
- xUnit
- build-your-own-testing-framework
- javascript
---

Today we are going to test-drive the testing framework without any external testing framework.
This will be done through test-driving a simple kata (FizzBuzzKata). For example:

- every time we expect a test to fail and it doesn't, this is a failing test for our testing framework, that we will be fixing,
- every time we expect a test to pass and it doesn't, this is another failing test for our testing framework, that we will be fixing.

For practical reasons, today we are going to use concrete programming language instead of pseudo-code - javascript. Except for small details, that we will point out, the techniques shown here are language-agnostic.

This article is only first one of the series "Build Your Own Testing Framework", so make sure to stick around for next parts!

Shall we begin?

## FizzBuzzKata

Given the number,

- return `Fizz` when the number is divisible by 3,
- return `Buzz` when the number is divisible by 5,
- return `FizzBuzz` when the number is divisible by 3 and 5,
- return string representation of number otherwise.

## Writing your first test

How do we write our first test, when we don't have a testing framework and we want to create one? - It seems, that we have to design how the test should like in our brand new testing framework.

I personally, would go with the xUnit-like design, since it is relatively simple. Given this, we might write our first test and it will look something like that:

```javascript
// test/FizzBuzzKataTest.js

function FizzBuzzKataTest() {
    this.testNormalNumberIsReturned = function() {
        this.assertTrue("1" === fizzBuzz(1));
    };
}
```

This test should fail, because function `fizzBuzz` is not defined, but it doesn't fail, since function `testNormalNumberIsReturned` is never called. In fact, the object with `FizzBuzzKataTest` is never being created.

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
        this.assertTrue("1" === fizzBuzz(1));
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
        this.assertTrue("1" === fizzBuzz(1));
             ^

TypeError: this.assertTrue is not a function
```

Clearly, to fix it we need to define `assertTrue` on `FizzBuzzKataTest` object. Obviously, we do not want our user to define all their assertion for every test suite. This means, that we want to define it on `FizzBuzzKataTest` object outside of the definition of `FizzBuzzKataTest`.

There are two ways to go about it:

- inheritance: make `FizzBuzzKataTest` inherit from some other object function `assertTrue`, or
- composition: make `FizzBuzzKataTest` accept a special object with function `assertTrue` defined on it.

I would like to go with composition method since it gives us more flexibility in the long run:

```javascript
function FizzBuzzKataTest(t) { ... }
```

and the usage of `assertTrue` has to change appropriately:

```javascript
    t.assertTrue("1" === fizzBuzz(1));
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
        t.assertTrue("1" === fizzBuzz(1), "Expected to equal " + "1" + ", but got: " + fizzBuzz(1));
    }

// ...

    assertTrue: function(condition, message) {
        throw new Error(message || "Expected to be true, bug got false");
    }
```

When running this, we are getting the expected error:

```
/path/to/project/test/FizzBuzzKataTest.js:11
        throw new Error(message || "Expected to be true, bug got false");
        ^

Error: Expected to equal 1, but got: undefined
```

This looks better now. Let's fix the error now by implementing the simplest thing that could work:

```javascript
function fizzBuzz(number) {
    return "1";
}
```

And as we run our test suite we get:

```
/path/to/project/test/FizzBuzzKataTest.js:13
        throw new Error(message || "Expected to be true, bug got false");
        ^

Error: Expected to equal 1, but got: 1
```

Oh, it should have passed the test. I know why it didn't: we throw this error unconditionally, let's add an appropriate `if` statement to `assertTrue` function:

```javascript
    assertTrue: function(condition, message) {
        if (!condition) {
            throw new Error(...);
        }
    }
```

If we run this code, it does not fail. That was our first green state - it took as awhile to get here. The reason for this is that we are not only test-driving `FizzBuzzKata`, additionally, we are writing a feature test for a non-existing testing framework. Now that we are green, we should think about refactoring, i.e.: making the structure of our code right.

Obviously, we should move our testing framework code outside of the test suite file. Probably, somewhere in `src/TestingFramework.js`. For that, we need to first parametrize `FizzBuzzKataTest` and extract the function to run the test suite.

Parametrize:

```javascript
var testSuiteConstructor = FizzBuzzKataTest;
var testSuite = new testSuiteConstructor(assertions);
testSuite.testNormalNumberIsReturned();
```

and extract method:

```javascript
function runTestSuite(testSuiteConstructor) {
    var testSuite = new testSuiteConstructor(assertions);
    testSuite.testNormalNumberIsReturned();
}

var testSuiteConstructor = FizzBuzzKataTest;
runTestSuite(testSuiteConstructor);
```

and inline the variable `testSuiteConstructor`:

```javascript
runTestSuite(FizzBuzzKataTest);
```

Now it is time to move testing code to `src/TestingFramework.js`:

```javascript
// src/TestingFramework.js

var assertions = {
    assertTrue: function (condition, message) {
        if (!condition) {
            throw new Error(message || "Expected to be true, bug got false");
        }
    }
};

function runTestSuite(testSuiteConstructor) {
    var testSuite = new testSuiteConstructor(assertions);
    testSuite.testNormalNumberIsReturned();
}
```

And to be able to require `runTestSuite` function:

```javascript
// src/TestingFramework.js

var assertions = { ... };

function runTestSuite(testSuiteConstructor) { ... }

module.exports = runTestSuite;
```

And, finally, let's use that from our test suite:

```javascript
// test/FizzBuzzKataTest.js

var runTestSuite = require("../src/TestingFramework");

function FizzBuzzKataTest(t) { ... }

function fizzBuzz(number) { ... }

runTestSuite(FizzBuzzKataTest);
```

If we run the test suite again, everything should pass. Somehow, I don't feel comfortable now, let's try to break the test suite and see if it will fail as expected:

```javascript
function fizzBuzz(number) {
    return "2";  // <-- "1" was changed to "2" here
}
```

And run tests:

```
/path/to/project/src/TestingFramework.js:4
            throw new Error(message || "Expected to be true, bug got false");
            ^

Error: Expected to equal 1, but got: 2
```

Yes, it still works as expected. We have just introduced a Mutation to our code, to see if it is still tested properly. Let's undo the Mutation and see the test still pass. And it does.

If you look closely now, it should be possible to inline `FizzBuzzKataTest` definition as an argument of `runTestSuite` call:

```javascript
var runTestSuite = require(...);

function fizzBuzz(number) { ... }

runTestSuite(function (t) { ... });
```

And if we run our test suite, it still works. Just to check, that we are still good, let's repeat our Mutation from the previous step. It should still fail as expected. And it does. Undo the mutation and the test is still passing. Great.

I think we are done with Refactoring step, for now, let's get back to writing another failing test.

## Writing the second test

```javascript
    this.testAnotherNormalNumberIsReturned = function() {
        t.assertTrue("2" === fizzBuzz(2), "Expected to equal " + "2" + ", but got: " + fizzBuzz(2));
    };
```

If we run these tests, they do not fail. This is strange, let's look at `runTestSuite` function again:

```javascript
function runTestSuite(testSuiteConstructor) {
    var testSuite = new testSuiteConstructor(assertions);
    testSuite.testNormalNumberIsReturned();
}
```

Great, it just runs one specific function, we should probably run all functions starting from `test` instead:

```javascript
function runTestSuite(testSuiteConstructor) {
    var testSuite = new testSuiteConstructor(assertions);

    for (var testName in testSuite) {
        if (testName.match(/^test/)) {
            testSuite[testName]();
        }
    }
}
```

*REMARK: this code is Javascript specific. Other programming languages will have their own way of iterating over the function/method list and calling a function by its name. Usually, it is some sort of reflection for compiled languages and meta-programming features for interpreted languages.*

If we run tests now, we get the expected failure:

```
/path/to/project/src/TestingFramework.js:4
            throw new Error(message || "Expected to be true, bug got false");
            ^

Error: Expected to equal 2, but got: 1
```

If we try to change `return "1"` to `return "2"`, of course this test will pass, but the other will fail:

```
/path/to/project/src/TestingFramework.js:4
            throw new Error(message || "Expected to be true, bug got false");
            ^

Error: Expected to equal 1, but got: 2
```

This is great for couple of reasons:

- It validates, that our change to how `test*` functions are discovered is correct, and
- We have to have a bit smarter implementation to pass both tests now:

```javascript
function fizzBuzz(number) {
    return number.toString();
}
```

And if we run the tests, they pass. Now, that we are in Green state, we should start refactoring. Have you noticed the duplication already?

```javascript
t.assertTrue("1" === fizzBuzz(1), "Expected to equal " + "1" + ", but got: " + fizzBuzz(1));

// and:

t.assertTrue("2" === fizzBuzz(2), "Expected to equal " + "2" + ", but got: " + fizzBuzz(2));
```

Extracting `"1"` and `"2"` as variable `expected`, and `fizzBuzz(1)` and `fizzBuzz(2)` as variable `actual`, makes these 2 lines identical:

```javascript
this.testNormalNumberIsReturned = function () {
    var expected = "1";
    var actual = fizzBuzz(1);
    t.assertTrue(expected === actual, "Expected to equal " + expected + ", but got: " + actual);
};

this.testAnotherNormalNumberIsReturned = function() {
    var expected = "2";
    var actual = fizzBuzz(2);
    t.assertTrue(expected === actual, "Expected to equal " + expected + ", but got: " + actual);
};
```

Specifically, this is identical:

```javascript
t.assertTrue(expected === actual, "Expected to equal " + expected + ", but got: " + actual);
```

This sounds like `t.assertEqual(expected, actual)` to me. So let's extract it:

```javascript
// src/TestingFramework.js

var assertions = {
    assertTrue: function(condition, message) { ... },

    assertEqual: function(expected, actual) {
        this.assertTrue(
          expected === actual,
          "Expected to equal " + expected + ", but got: " + actual
        );
    }
}
```

Now, let's use it and inline our `expected` and `actual` variables:

```javascript
this.testNormalNumberIsReturned = function () {
    t.assertEqual("1", fizzBuzz(1));
};

this.testAnotherNormalNumberIsReturned = function() {
    t.assertEqual("2", fizzBuzz(2));
};
```

This looks much more readable. If we run tests, they still pass. If we try to break our code by using some Mutation, the tests fail as expected. Great, our refactoring was a success!

Let's finish test-driving our `fizzBuzz` function.

## Test-Driving Fizz Buzz Kata

First test for `Fizz` case:

```javascript
// test:
this.testFizzIsReturned = function () {
    t.assertEqual("Fizz", fizzBuzz(3));
};

// Error: Expected to equal Fizz, but got: 3

// implementation:
function fizzBuzz(number) {
    if (number === 3) return "Fizz";
    return number.toString();
}
```

This is pretty stupid implementation, but it works for that one tests, so let's write the test, that will break this implementation and force us to write real `if` condition:

```javascript
// test:
this.testFizzIsReturnedForDifferentNumber = function () {
    t.assertEqual("Fizz", fizzBuzz(6));
};

// Error: Expected to equal Fizz, but got: 6

// implementation:
function fizzBuzz(number) {
    if (number % 3 === 0) return "Fizz";
    return number.toString();
}
```

This technique is called Triangulation:

- the first test is to force us to write some `if` statement with a correct body,
- second is to force us to make the condition right.
- If we had an `else` clause, we would have had another test to make that part right.

OK, that looks like a right implementation for `Fizz`, let's write the test for `Buzz` now:

```javascript
// test:
this.testBuzzIsReturned = function () {
    t.assertEqual("Buzz", fizzBuzz(5));
};

// Error: Expected to equal Buzz, but got: 5

// stupid implementation:
function fizzBuzz(number) {
    if (number === 5) return "Buzz";
    if (number % 3 === 0) return "Fizz";
    return number.toString();
}

// Triangulation:
this.testBuzzIsReturnedForDifferentNumber = function () {
    t.assertEqual("Buzz", fizzBuzz(10));
};

// Error: Expected to equal Buzz, but got: 10

// correct implementation:
function fizzBuzz(number) {
    if (number % 5 === 0) return "Buzz";
    if (number % 3 === 0) return "Fizz";
    return number.toString();
}
```

And finally let's implement final requirement `FizzBuzz`:

```javascript
// test:
this.testFizzBuzzIsReturned = function () {
    t.assertEqual("FizzBuzz", fizzBuzz(15));
};

// Error: Expected to equal FizzBuzz, but got: Buzz

// stupid implementation:
function fizzBuzz(number) {
    if (number === 15) return "FizzBuzz";
    if (number % 5 === 0) return "Buzz";
    if (number % 3 === 0) return "Fizz";
    return number.toString();
}

// Triangulation:
this.testFizzBuzzIsReturnedForDifferentNumber = function () {
    t.assertEqual("FizzBuzz", fizzBuzz(30));
};

// Error: Expected to equal FizzBuzz, but got: Buzz

// correct implementation:
function fizzBuzz(number) {
    if (number % 15 === 0) return "FizzBuzz";
    if (number % 5 === 0) return "Buzz";
    if (number % 3 === 0) return "Fizz";
    return number.toString();
}
```

I think we are done with the implementation. `FizzBuzzKata` has an extended set of requirements, but they are out of the scope of this article. These requirements force us to introduce Strategy pattern and stop using this unmaintainable chain of `if` statements.

Refactoring this code to Strategy pattern is left as an exercise for the reader.

## Bottom Line

Congratulations! Using `FizzBuzzKata` we have test-driven bare-bones testing framework to the point, that we can do Test-Driven Development for a simple Kata. And all that without having any testing framework in place.

The code is available on Github: https://github.com/waterlink/BuildYourOwnTestingFrameworkPart1

Now, with this minimal framework in place, it should be possible to unit-test the framework itself, so that we can support more use cases. This will be covered in next series of "Build Your Own Testing Framework". Stay tuned!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
