---
layout: post
title: "Build Your Own Testing Framework. Part 5"
description: "Did you notice, that out testing framework quits on the first failure? It probably should run all tests, collect all failures and present them nicely. This is what we are going to accomplish today..."
date: 2016-11-13 12:50:51 +0100
comments: true
categories:
- build-your-own-testing-framework
- javascript
- tdd
- test-driven-development
- testing
- xunit
---

Welcome back to the new issue of "Build Your Own Testing Framework" series! Did you notice, that out testing framework quits on the first failure? It probably should run all tests, collect all failures and present them nicely. This is what we are going to accomplish today:

- Make sure all tests run even when there is a failure.
- Make sure exit code is correct.

<!-- more -->

This article is the fifth one of the series “Build Your Own Testing Framework” so make sure to stick around for next parts! Find all posts of these series can [here](/blog/categories/build-your-own-testing-framework/).

Shall we get started?

## Catch and report a test failure

Our test suite should no longer bubble up any exceptions. We can achieve that by making an appropriate assertion. And also we should verify that other tests execute after the failure:

```javascript
runTestSuite(function FailureTest(t) {
    this.testItDoesNotBubbleUpExceptions = function () {
        var aSpy = t.spy();

        t.assertNotThrow(function () {
            runTestSuite(function (t) {
                this.testFailure = function () {
                    t.assertTrue(false);
                };

                this.testSomething = aSpy;
            });
        });

        aSpy.assertCalled();
    };
});
```

As expected, this fails with an appropriate error `Error: Expected not to throw error, but thrown 'Expected to be true, but got false'` indicating that we are bubbling up all errors at the moment. Also, notice how the execution of the whole test suite stops at that point, and it just exits the program with error code `1`. A simple `try .. catch` block will fix the issue:

```javascript
// in runTestSuite function
    for (var testName in testSuitePrototype) {
        if (testName.match(/^test/)) {
            reporter.reportTest(testName);
            var testSuite = createTestSuite(testSuiteConstructor);

            try {
                testSuite[testName]();
            } catch (error) {
                // do nothing, for now
            }
        }
    }
```

All tests now run successfully. This code is starting to become unreadable, so it is a good point to refactor. We will:

- Extract whole `try .. catch` as a function `runTest`. Its current responsibility is only to run the test and ignore any failure;
- Extract contents of `if` statement that matches the test name as a function `handleTest`. Its responsibility is to report the test, create a fresh testSuite and kick off `runTest`;
- Extract the whole `for` statement as `runAllTests`.

Here is the final snippet of code:

```javascript
function runTest(testSuite, testName) {
    try {
        testSuite[testName]();
    } catch (error) {
        // do nothing, for now
    }
}

function handleTest(reporter, testName, testSuiteConstructor) {
    reporter.reportTest(testName);
    runTest(createTestSuite(testSuiteConstructor), testName);
}

function runAllTests(reporter, testSuitePrototype, testSuiteConstructor) {
    for (var testName in testSuitePrototype) {
        if (testName.match(/^test/)) {
            handleTest(reporter, testName, testSuiteConstructor);
        }
    }
}

function runTestSuite(testSuiteConstructor, options) {
    options = options || {};
    var reporter = options.reporter || new SimpleReporter();

    var testSuitePrototype = createTestSuite(testSuiteConstructor);

    reporter.reportTestSuite(
        getTestSuiteName(testSuiteConstructor, testSuitePrototype)
    );

    runAllTests(reporter, testSuitePrototype, testSuiteConstructor);
}
```

## Exit with code 1

Now, when at least one test fails in a suite of tests, the whole suite should fail (after running the rest of its tests). And the indicator of such failure should be an exit code of the process. Let's write a test:

```javascript
runTestSuite(function FailureTest(t) {
    // ...

    this.testItExitsWithProcessCodeOne = function () {
        var processSpy = new ProcessSpy();

        runTestSuite(function (t) {
            this.testFailure = function () {
                t.assertTrue(false);
            };
        }, {process: processSpy});

        t.assertEqual(1, processSpy.hasExitedWithCode);
    };
});
```

As you might guess, we will need another object. It will be responsible for interaction with our process, i.e.: something that we can ask to "exit with code 1." Because we can not ask our process to exit within the test run, we will have to create a spy. And we shall test-drive its functionality. There is something interesting that we should worry about before that - our test suite is passing currently.. but it shouldn't be!

Let's step back and think what just happened: clearly, we are writing the test, that can not possibly pass because we do not have `ProcessSpy` yet. So we are expecting a failure - we are expecting a thrown exception. That expectation is an important part of test-driven development: at all times we expect a very specific failure or we expect our tests to pass; if we do not receive a failure when expected and receive an unexpected failure, we should stop right there and think which part of our thinking and our assumptions is incorrect.

{% include v2/screencast-subscribe.html %}

Right now, tests do not fail, because we are ignoring all exceptions in our `try .. catch` that we introduced a couple of minutes ago. If we want to see failures again, let's modify `catch` block to just log all errors it receives:

```javascript
function runTest(testSuite, testName) {
    try {
        testSuite[testName]();
    } catch (error) {
        console.log(error);
    }
}
```

Now our test suite outputs an expected error: `ReferenceError: ProcessSpy is not defined`. Also, it outputs some other failures that happen in our nested `runTestSuite` calls - we should fix them by providing `silenceFailures` option for nested `runTestSuite` call. We can focus now on the `ProcessSpy` failure and test-drive it:

```javascript
runTestSuite(function ProcessSpy_BehaviorTest(t) {
    var processSpy = new ProcessSpy();
});
// => ReferenceError: ProcessSpy is not defined

function ProcessSpy() {}
// => PASS

    this.testHasExitedWithCode_initiallyIsNull = function () {
        t.assertEqual(null, processSpy.hasExitedWithCode);
    };
// => Error: Expected to equal null, but got: undefined

function ProcessSpy() {
    this.hasExitedWithCode = null;
}
// => PASS

    this.testHasExitedWithCode_isZero_afterExitZeroCall = function () {
        processSpy.exit(0);
        t.assertEqual(0, processSpy.hasExitedWithCode);
    };
// => TypeError: processSpy.exit is not a function

// in ProcessSpy
    this.exit = function (code) {
        this.hasExitedWithCode = 0;
    };
// => PASS

    this.testHasExitedWithCode_isOne_afterExitOneCall = function () {
        processSpy.exit(1);
        t.assertEqual(1, processSpy.hasExitedWithCode);
    };
// => Error: Expected to equal 1, but got: 0

// in ProcessSpy
    this.exit = function (code) {
        this.hasExitedWithCode = code;
        // changed 0 to code      ^here^
    };
```

I think we have finished test-driving the functionality of `ProcessSpy`. It is time to get back to our failing test for a failure resulting in an exit with code 1. When we run this test suite, we are getting the following error message: `Error: Expected to equal 1, but got: null`.' To pass this test, we will need to store the fact that we had a failure somewhere and at the end of the test suite run we can trigger exit with code 1 or 0, respectively. We could pass around a `status` object with boolean property `status.failed` and set it to `true` in our `catch` block:

```javascript
    } catch (error) {
        if (!silenceFailures) console.log(error);
        status.failed = true;
    }
```

And at the end of `runTestSuite` function we could call `process.exit(1)` if `status.failed` was `true`:

```javascript
function runTestSuite(testSuiteConstructor, options) {
    // ...

    if (status.failed) {
        process.exit(1);
    }
}
```

While this works (as in "tests pass after providing `fakeProcess` where needed for nested failing `runTestSuite` calls") state changes in this code are starting to be hard to follow and function signatures remind me of some horror movie:

```javascript
function getTestSuiteName(testSuiteConstructor, testSuitePrototype)
function runTest(testSuite, testName, silenceFailures, status)
function handleTest(reporter, testName, testSuiteConstructor, silenceFailures, status)
function runAllTests(reporter, testSuitePrototype, testSuiteConstructor, silenceFailures, status)
```

These signatures smell like objects are hiding there in these functions. Let's find them!

## Quest for hidden objects

First, let's extract the method object from the function `runTestSuite`. We will give it a name `TestSuiteRunContext`:

```javascript
function TestSuiteRunContext(testSuiteConstructor, options) {
    options = options || {};
    var reporter = options.reporter || new SimpleReporter();
    var process = options.process || global.process;
    var silenceFailures = options.silenceFailures || false;

    var status = {failed: false};

    var testSuitePrototype = createTestSuite(testSuiteConstructor);

    this.invoke = function () {
        reporter.reportTestSuite(
            getTestSuiteName(testSuiteConstructor, testSuitePrototype)
        );

        runAllTests(
            reporter,
            testSuitePrototype,
            testSuiteConstructor,
            silenceFailures,
            status
        );

        if (status.failed) {
            process.exit(1);
        }
    };
}

function runTestSuite(testSuiteConstructor, options) {
    new TestSuiteRunContext(testSuiteConstructor, options).invoke();
}
```

Now, if we were to move function `runAllTests` inside of this class, we would not need all these arguments (and all other functions we call):

```javascript
    this.invoke = function () {
        reportTestSuite();
        runAllTests();
        finishTestRun();
    };

    function reportTestSuite() {
        reporter.reportTestSuite(getTestSuiteName());
    }

    function getTestSuiteName() {
        if (typeof(createTestSuite().getTestSuiteName) !== "function") {
            return testSuiteConstructor.name;
        }

        return createTestSuite().getTestSuiteName();
    }

    function createTestSuite() {
        return new testSuiteConstructor(assertions);
    }

    function runAllTests() {
        for (var testName in createTestSuite()) {
            if (testName.match(/^test/)) {
                handleTest(testName);
            }
        }
    }

    function handleTest(testName) {
        reportTest(testName);
        runTest(createTestSuite(), testName);
    }

    function reportTest(testName) {
        reporter.reportTest(testName);
    }

    function runTest(testSuite, testName) {
        try {
            testSuite[testName]();
        } catch (error) {
            if (!silenceFailures) console.log(error);
            status.failed = true;
        }
    }

    function finishTestRun() {
        if (status.failed) {
            process.exit(1);
        }
    }
```

It already looks very nice. The only thing that I do not like about this object yet is that it has stateful properties and stateless properties. I like to have my objects separated by this concern. Let's extract `status` mutable property as a proper `TestSuiteRunStatus` object:

```javascript
function TestSuiteRunStatus() {
    var failed = false;

    this.markAsFailed = function () {
        failed = true;
    };

    this.hasFailed = function () {
        return failed;
    };
}

function TestSuiteRunContext(testSuiteConstructor, options) {
  // ...
  var status = new TestSuiteRunStatus();

  // ...
  function runTest(testSuite, testName) {
        try {
            testSuite[testName]();
        } catch (error) {
            if (!silenceFailures) console.log(error);
            status.markAsFailed();
        }
    }

    function finishTestRun() {
        if (status.hasFailed()) {
            process.exit(1);
        }
    }
}
```

I think we have finished the refactoring. Now we should verify that test suite exits with the code 0 when everything passes:

```javascript
    this.testItExitsWithProcessCodeZero_onSuccess = function () {
        runTestSuite(function (t) {
            this.testFailure = function () {
                t.assertTrue(true);
            };
        }, {process: processSpy, silenceFailures: true});

        t.assertEqual(0, processSpy.hasExitedWithCode);
    };
// => Error: Expected to equal 0, but got: null

    function finishTestRun() {
        if (status.hasFailed()) return process.exit(1);
        process.exit(0);
    }
// => PASS
```

## Bottom Line

I think we have finished implementing exit code reporting. The code can be found here: https://github.com/waterlink/BuildYourOwnTestingFrameworkPart5

There is still a lot to go through. In a few next episodes we will:

- Report OK and FAIL for each test;
- Output carefully formatted failures to the STDERR;
- Enable our testing framework to run multiple test suite files at once;
- Enable our testing framework to run in a browser (it is javascript after all).

Stay tuned!

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on Twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don’t hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
