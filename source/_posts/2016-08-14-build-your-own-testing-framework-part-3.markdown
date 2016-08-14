---
layout: post
title: "Build Your Own Testing Framework. Part 3"
date: 2016-08-14 13:55:17 +0200
comments: true
categories:
- TDD
- test-driven-development
- testing
- xUnit
- build-your-own-testing-framework
- javascript
---

Welcome back to the new issue of "Build Your Own Testing Framework" series! Today we are going to unit-test `runTestSuite` function of our testing framework. Currently, only its happy path is implicitly tested via every test of the system. Additionally, this function's unhappy paths are, in fact, untestable at the moment.

This article is the third one of the series "Build Your Own Testing Framework", so make sure to stick around for next parts! All articles of these series can be found [here](/blog/categories/build-your-own-testing-framework/).

Shall we get started?

## Testing existing code

Let's take another look at the `runTestSuite` function:

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

This function currently:

1. Creates a new test suite from the passed in function-constructor.
2. Finds every method that starts with the string `test`.
3. And calls every such method.

### Test that it calls at least one test method

```javascript
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function (t) {
    this.testItCallsOneTestMethod = function () {
        var called = false;

        runTestSuite(function (t) {
            this.testSomeInterestingFunction = function () {
                called = true;
            };
        });

        t.assertTrue(called);
    };
});
```

And this test passes. To be sure, that we are actually testing anything, let's make sure that `testSomeInterestingFunction` is not being called:

```javascript
if (testName.match(/^test/) &&
    testName != "testSomeInterestingFunction") {
//  ^   make sure our method is not called  ^
    testSuite[testName]();
}
```

This fails as expected: `Error: Expected to be true, but got false`. Undoing this mutation causes the test to pass again. This is good, since we have seen this test fail when we expect it to fail. This proves the semantic stability of our test.

So, what will happen if we replace the whole `if` condition with `true`?

```javascript
if (true) {
    testSuite[testName]();
}
```

As expected, all tests will pass. Seems that we need to add a new test here:

### Test that it does not call non-test methods

```javascript
this.testItDoesNotCallMethodThatDoesNotStartWithTestPrefix = function () {
    var called = false;

    runTestSuite(function (t) {
        this.someFunction = function () {
            called = true;
        };
    });

    t.assertTrue(!called);
};
```

And this fails as expected. This makes our test suite semantically stable against this sort of mutation. Undoing the mutation should make the test suite pass. And it does.

There is another surviving mutant that I can come up with:

```javascript
for (var testName in testSuite) {
    if (testName.match(/^test/)) {
        testSuite[testName]();
    }
    break; // <- surviving mutant
}
```

This means, that only first function will only ever run. Since all our tests are currently verifying that only one function is called or not we will need another test to defeat this mutant:

### Test that it calls all provided test methods

```javascript
this.testItCallsAllTestMethods = function () {
    var calledOne = false;
    var calledTwo = false;
    var calledThree = false;

    runTestSuite(function (t) {
        this.testFunctionOne = function () {
            calledOne = true;
        };

        this.testFunctionTwo = function () {
            calledTwo = true;
        };

        this.testFunctionThree = function () {
            calledThree = true;
        };
    });

    t.assertTrue(calledOne);
    t.assertTrue(calledTwo);
    t.assertTrue(calledThree);
};
```

Careful here: `testItCallsAllTestMethods` has to be the first test in the test suite for it to be ever called with the current mutation. As expected this test fails and undoing the mutation makes it pass.

`testItCallsAllTestMethods` is superior to the `testItCallsOneTestMethod`, so we can remove the latter.

The amount of duplication in this code does not make me happy. Seems like we are missing the ability to verify if a certain function was called or not. Let's try to extract this abstraction:

```javascript
function spy() {
    function that() {
        that.called = true;
    }

    that.called = false;

    return that;
};
```

And then the usage would look like that:

```javascript
this.testItCallsAllTestMethods = function () {
    var spyOne = spy();
    var spyTwo = spy();
    var spyThree = spy();

    runTestSuite(function (t) {
        this.testFunctionOne = spyOne;
        this.testFunctionTwo = spyTwo;
        this.testFunctionThree = spyThree;
    });

    t.assertTrue(spyOne.called);
    t.assertTrue(spyTwo.called);
    t.assertTrue(spyThree.called);
};

this.testItDoesNotCallMethodThatDoesNotStartWithTestPrefix = function () {
    var aSpy = spy();

    runTestSuite(function (t) {
        this.someFunction = aSpy;
    });

    t.assertTrue(!aSpy.called);
};
```

## Testing `assertions.spy()`

It seems, that having `t.spy()` available for the users of our testing framework might be very useful! Let's test-drive it:

```javascript
// Initially, it should not be called
this.testIsNotCalledInitially = function () {
    t.assertTrue(!t.spy().called);
};

// TypeError: t.spy is not a function

// Implementation in `assertions object`:
spy: function () {
    return {
        called: false
    }
}

// Let's check that it can be called as a function
this.testItCanBeCalledAsFunction = function () {
    t.spy()();
};

// TypeError: t.spy(...) is not a function

// Simplest implementation:
spy: function () {
    return function () {};
}

// Let's check that after being called it has correct `.called` value
this.testIsCalledAfterBeingCalled = function () {
    var aSpy = t.spy();
    aSpy();
    t.assertTrue(aSpy.called);
};

// Error: Expected to be true, but got false

// And final implementation:
spy: function () {
    return function that() {
        that.called = true;
    };
}
```

Test `testItCanBeCalledAsFunction` is inferior to `testIsCalledAfterBeingCalled`, so we can remove it.

To make the assertion more fluent we might want to have `aSpy.assertCalled()` and `aSpy.assertNotCalled()`:

```javascript
// Let's replace our first test's `assertTrue(!...)` with `.assertNotCalled()`
this.testIsNotCalledInitially = function () {
    t.spy().assertNotCalled();
};

// TypeError: t.spy(...).assertNotCalled is not a function

// And the stupid implementation:
spy: function () {
    function that() {
        that.called = true;
    }

    that.assertNotCalled = function () {};

    return that;
}

// This needs some triangulation:
this.testAssertNotCalledFailsWhenWasCalled = function () {
    var aSpy = t.spy();
    aSpy();

    t.assertThrow("Expected not to be called", function () {
        aSpy.assertNotCalled();
    });
};

// Error: Expected to throw an error, but nothing was thrown

// And to make it pass:
that.assertNotCalled = function () {
    assertions.assertTrue(!that.called, "Expected not to be called");
};
```

Let's do the same with the other test:

```javascript
// Replace `assertTrue` in the second test with `assertCalled`:
this.testIsCalledAfterBeingCalled = function () {
    var aSpy = t.spy();
    aSpy();
    aSpy.assertCalled();
};

// TypeError: aSpy.assertCalled is not a function

// And the stupid implementation:
that.assertCalled = function () {};

// Let's triangulate it a bit:
this.testAssertCalledFailsWhenWasNotCalled = function () {
    t.assertThrow("Expected to be called", function () {
        t.spy().assertCalled();
    });
};

// Error: Expected to throw an error, but nothing was thrown

// And the implementation:
that.assertCalled = function () {
    assertions.assertTrue(that.called, "Expected to be called");
};
```

And the full implementation of `spy()` function:

```javascript
spy: function () {
    function that() {
        that.called = true;
    }

    that.assertNotCalled = function () {
        assertions.assertTrue(!that.called, "Expected not to be called");
    };

    that.assertCalled = function () {
        assertions.assertTrue(that.called, "Expected to be called");
    };

    return that;
}
```

## Bottom Line

Today we have tested all the existing behavior of the `runTestSuite` function. That has driven us to implement very simple spies for our testing framework.

We have successfully applied manual Mutational Testing to the existing functionality to derive Semantically Stable tests for it. We did some Triangulation too today. Generally speaking, in TDD Triangulation technique is something that is used on a daily basis, when TDD is practiced properly. Future articles will expand on Triangulation, Mutational Testing, and Semantic Stability in more detail, so stay tuned!

There are only a few problems left, that bother me:

- We often do `t.assertTrue(!condition)`, seems that we lack `t.assertFalse(condition)` assertion.
- We often call functions without any assertions to do an implicit assertion, that the call is not throwing any exception. This can be confusing: it is better to make that assertion explicitly - seems like we need `t.assertNotThrow`.
- Seems that it is useful to have `NOT` version of every assertion. Even though we don't need `t.assertNotEqual` right now, from my experience with testing it is often very useful.

Creating these assertions I will leave as an exercise to the reader. From now on, we will assume they are implemented and we will use them where appropriate. Code is available on GitHub: https://github.com/waterlink/BuildYourOwnTestingFrameworkPart3

Next time we will add more requirements for our `runTestSuite` function, such as:

- Continue running tests after the first failure.
- Report successfully passed tests.
- Report failures.
- Report test run stats (counts of successful and failed tests).
- Avoid shared state between tests of the same test suite.

Stay tuned!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
