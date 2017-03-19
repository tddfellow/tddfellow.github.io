---
layout: post
title: "Build Your Own Testing Framework. Part 6: Test Suite does not Run All Tests!"
date: 2017-03-13 23:13:28 +0100
description: "When trying to implement better formatting, we have discovered that some of our test suites do not run all tests! Today we are going to fix that, and we will make sure that such test suite will fail if it didn't execute all tests."
comments: true
categories:
- build-your-own-testing-framework
- javascript
- tdd
- test-driven-development
- testing
- xunit
---


Welcome back to the new issue of "Build Your Own Testing Framework" series! When trying to implement better formatting, we have discovered that some of our test suites do not run all tests! Today we are going to fix that, and we will make sure that such test suite will fail if it didn't execute all tests.

<!-- more -->

This article is the sixth one of the series “Build Your Own Testing Framework” so make sure to stick around for next parts! Find all posts of these series [here](/blog/categories/build-your-own-testing-framework/).

Shall we get started?

## Verify All Tests Run

We will start from the `RunTestSuiteTest` and run the test suite with the single test. Then we are going to assert that for that the test with the name `testOk` has been reported as passing:

```javascript
// test/RunTestSuiteTest.js

this.testItOutputsOkForThePassingTest = function () {
    runTestSuite(function (t) {
        this.testOk = function () {
            t.assertTrue(true);
        };
    }, {reporter: reporter});

    reporter.assertHasReportedPassingTest("testOk");
};
```

If we run this test suite, we can see that only one test executes!

```
RunTestSuiteTest
    testItCallsAllTestMethods

Process finished with exit code 0
```

Oh, that is interesting. This test suite does not run. Upon investigating, it turns out, that `process.exit(0)` is being called during the `runTestSuite(...)` function run. That is because of the latest feature that we have implemented - "exit with an appropriate exit code (zero for success, and one for failure)." We should be able to fix that by providing the process spy in the options of the `runTestSuite` function that we are calling from the inside of the individual tests in the `RunTestSuiteTest` test suite. And we ought to alleviate this kind of mistake somehow - we need a mechanism that would alert us if not all tests have been run. Maybe something like `verifyAllTestsRun: true` option for the `runTestSuite`. For that let's write a test:

```javascript
this.testVerifyAllTestsRun = function () {
    t.assertThrow("Expected all tests to run", function () {

        runTestSuite(function SuiteWithTwoTests(t) {

            this.testWithRunTestSuite = function () {
                runTestSuite(function (t) {
                    this.testOne = function () {};
                }, {reporter: reporter});
            };

            this.testThatShouldAlsoRun = function () {};

        }, {reporter: reporter,
            process: process,
            verifyAllTestsRun: true});

    });
};
```

That might be a bit complex at first. Let's take a closer look how this test is supposed to work:

1. First of all, we do assert that there was an assertion failure about all tests required to run.
2. Inside of the action for this assertion we create and run the new test suite with two tests:
    - test with the `runTestSuite` without process spy provided
    - empty test that should also execute

If we run this test, it will pass. That is unexpected because we wanted it to fail. Apparently, most inner `runTestSuite` is doing `process.exit(0)`.

For that to work, we will need to be able to provide a hook into `process.exit(code)` function. For that, we would need to create a `SimpleProcess` class, that allows installation of such hooks. Let's test-drive it!

### `process.exit` with hooks

First, we should start from the normal behavior without any hooks:

```javascript
// test/SimpleProcessTest.js
var runTestSuite = require("../src/TestingFramework");
var SimpleProcess = require("../src/TestingFramework").SimpleProcess;
var ProcessSpy = require("./ProcessSpy");

runTestSuite(function SimpleProcessTest(t) {
    this.testWithoutHooks = function () {
        var globalProcess = new ProcessSpy();
        var process = new SimpleProcess(globalProcess);

        process.exit(0);

        t.assertEqual(0, globalProcess.hasExitedWithCode);
    };
});
```

When running this test, we will get a failure about `SimpleProcess` being undefined. So let's define it:

```javascript
// src/TestingFramework.js

// ...
// define the class itself
function SimpleProcess(globalProcess) {
    
}

// ..
// and don't forget to export it
module.exports.SimpleProcess = SimpleProcess;
```

If we run our test suite now, we will get an error `TypeError: process.exit is not a function`. To fix that failure we will have to define the `exit(code)` method on our newly created class `SimpleProcess`:

```javascript
function SimpleProcess(globalProcess) {
    this.exit = function (code) {
        
    };
}
```

After doing that we will get an assertion failure `Error: Expected to equal 0, but got: null`, as expected. To make the test pass, it would be enough to call `globalProcess.exit(0)`:

```javascript
this.exit = function (code) {
    globalProcess.exit(0);
};
```

If we run our test suite now, we will get no failures. That is great! Now, we can see that `globalProcess.exit(0)` is probably not exactly what we want to have there. We ought to pass the `code` parameter to the `exit` function. To test-drive this properly, we will have to triangulate, i.e.: add another test with the different value of the `code` parameter:

```javascript
this.testWithoutHooks_andDifferentExitCode = function () {
    var globalProcess = new ProcessSpy();
    var process = new SimpleProcess(globalProcess);

    process.exit(1);

    t.assertEqual(1, globalProcess.hasExitedWithCode);
};
```

That fails as expected: `Error: Expected to equal 1, but got: 0`. To make it pass we can either write some weird "if" statement or we could pass the `code` parameter to the `globalProcess.exit` function. The second option is simpler. According to the third rule of test-driven development, we should go for it:

```javascript
this.exit = function (code) {
    globalProcess.exit(code);
};
```

That change makes our test suite pass. We probably should refactor the test suite to reduce the level of the duplication by extracting common variables from the tests:

```javascript
runTestSuite(function SimpleProcessTest(t) {
    var globalProcess = new ProcessSpy();
    var process = new SimpleProcess(globalProcess);

    this.testWithoutHooks = function () {
        process.exit(0);

        t.assertEqual(0, globalProcess.hasExitedWithCode);
    };

    this.testWithoutHooks_andDifferentExitCode = function () {
        process.exit(1);

        t.assertEqual(1, globalProcess.hasExitedWithCode);
    };
});
```

At that point, we should move on to tests for the hook installation functionality. Because right now we need only at most one hook we will not support multiple hooks at the same time - only one:

```javascript
this.testCanInstallOneHook = function () {
    var aSpy = t.spy();

    process.installHook(aSpy);
    process.exit(0);

    aSpy.assertCalled();
};
```

When we run this test, it fails because `installHook` function is not defined: `TypeError: process.installHook is not a function`. So we should define it:

```javascript
function SimpleProcess(globalProcess) {
    // ..
    this.installHook = function (aHook) {

    };
}
```

Upon running these tests, we get `Error: Expected to be called` because we didn't call this hook yet. The simplest way to make it pass is to just call the hook from the `installHook` function:

```javascript
function SimpleProcess(globalProcess) {
    // ...
    this.installHook = function (aHook) {
        aHook();
    };
}
```

While that will make the tests pass it is not the behavior that we are after. To drive out the correct behavior, we ought to check that the function is being called only after `process.exit(..)`, not earlier. For that we will need to have a sanity-check assertion:

```javascript
this.testCanInstallOneHook = function () {
    var aSpy = t.spy();

    process.installHook(aSpy);
    aSpy.assertNotCalled();

    process.exit(0);

    aSpy.assertCalled();
};
```

That fails as expected with the error `Error: Expected not to be called`. To make it pass we need to store the function in the variable and call it from the `process.exit(..)`:

```javascript
function SimpleProcess(globalProcess) {
    var hook = null;

    this.exit = function (code) {
        if (hook !== null)
            hook();
        
        globalProcess.exit(code);
    };

    this.installHook = function (aHook) {
        hook = aHook;
    };
}
```

All the tests pass now! Finally, we want to be able to uninstall the hook, so let's write the test for it:

```javascript
this.testCanUninstallTheHook = function () {
    var aSpy = t.spy();

    process.installHook(aSpy);
    process.uninstallHook();

    process.exit(0);

    aSpy.assertNotCalled();
};
```

To make it work it is enough to introduce this function and set `hook` variable back to `null` in it:

```javascript
function SimpleProcess(globalProcess) {
    // ...
    this.uninstallHook = function () {
        hook = null;
    };
}
```

And all the tests will pass. Now we, also, want to replace the default value for the `options.process` option with the instance of `SimpleProcess` object. And all the tests should work as they were working before:

```javascript
var simpleProcess = new SimpleProcess(global.process);

function TestSuiteRunContext(testSuiteConstructor, options) {
    // ...
    
    var process = options.process || simpleProcess;
    // instead of just "global.process"
    
    // ...
}
```

### Installing the "verify all tests run" hook

Now, we can get back to our "verify all tests run" test. It still doesn't fail as expected, so we need to install the hook, count all tests, count tests that had already run and compare them in the hook:

```javascript
function TestSuiteRunContext(testSuiteConstructor, options) {
    // ...
    var verifyAllTestsRun = options.verifyAllTestsRun || false;
    var testCount = 0;
    var testRun = 0;
    
    this.invoke = function () {
        if (verifyAllTestsRun)
            installVerifyAllTestsRunHook();  // <---

        reportTestSuite();
        countAllTests();                     // <---
        runAllTests();
        finishTestRun();
    };
    
    function installVerifyAllTestsRunHook() {
        simpleProcess.installHook(function () {
            if (testRun < testCount) {
                throw new Error("Expected all tests to run");
            }
        });
    }
    
    // ...
    
    function countAllTests() {
        for (var testName in createTestSuite())
            if (testName.match(/^test/))
                testCount++;
    }
    
    // ...
    
    function handleTest(testName) {
        testRun++;                            // <---
        reportTest(testName);
        runTest(createTestSuite(), testName);
    }
}
```

At this point, this throws an error `Expected all tests to run` and finishes the test fully without reaching our `assertThrow(..)` assertion. That happens because we catch this error in the function `runTest`, where we mark the test as failed, log the error and ignore the error object itself from there. One way to solve this problem is to have a particular error, that can propagate up the stack:

```javascript
function installVerifyAllTestsRunHook() {
    simpleProcess.installHook(function () {
        if (testRun < testCount) {
            var error = new Error("Expected all tests to run");
            error.bubbleUp = true;
            throw error;
        }
    });
}

// ...

function runTest(testSuite, testName) {
    try {
        testSuite[testName]();
    } catch (error) {
        if (error.bubbleUp) throw error;  // <---
        if (!silenceFailures) console.log(error);
        status.markAsFailed();
    }
}
```

Now our current test is passing, and the next test is failing with the error `Expected all tests to run`. That happens because we have not uninstalled the hook as soon as it has triggered. Let's do that:

```javascript
function installVerifyAllTestsRunHook() {
    simpleProcess.installHook(function () {
        if (testRun < testCount) {
            simpleProcess.uninstallHook();   // <---
            
            var error = new Error("Expected all tests to run");
            error.bubbleUp = true;
            throw error;
        }
    });
}
```

That makes the next test run, succeed and exit immediately after that with error code zero. Let's see what will happen if we put `verifyAllTestsRun: true` on the top test suite here:

```javascript
runTestSuite(function RunTestSuiteTest(t) {
    // ...
}, {verifyAllTestsRun: true});
```

That doesn't work because we re-install different hook inside of this test and as soon as this test finishes, we uninstall it. So we have two ways out of this situation: allow multiple hooks, or move that single test to its own test suite file. I think the second options is much simpler. Also, we will add the test for the negative case, where all tests run correctly (when we provide proper process spy):

```javascript
// test/VerifyAllTestsRunTest.js
var runTestSuite = require("../src/TestingFramework");
var ReporterSpy = require("./ReporterSpy");
var ProcessSpy = require("./ProcessSpy");

runTestSuite(function VerifyAllTestsRunTest(t) {
    var reporter = new ReporterSpy(t);
    var process = new ProcessSpy(t);

    this.testVerifyAllTestsRun = function () {
        t.assertThrow("Expected all tests to run", function () {

            runTestSuite(function SuiteWithTwoTests(t) {

                this.testWithRunTestSuite = function () {
                    runTestSuite(function (t) {
                        this.testOne = function () {};
                    }, {reporter: reporter});
                };

                this.testThatShouldAlsoRun = function () {};

            }, {reporter: reporter,
                process: process,
                verifyAllTestsRun: true});

        });
    };

    this.testVerifyAllTestsRun_withoutFailure = function () {
        t.assertNotThrow(function () {

            runTestSuite(function SuiteWithTwoTests(t) {

                this.testWithRunTestSuite = function () {
                    runTestSuite(function (t) {
                        this.testOne = function () {};
                    }, {reporter: reporter,
                        process: process});
                };

                this.testThatShouldAlsoRun = function () {};

            }, {reporter: reporter,
                process: process,
                verifyAllTestsRun: true});

        });
    };
});
```

And this new test suite passes as expected. Just to double-check that these tests verify anything, we can break them (change expected error message and change `assertNotThrow` to `assertThrow`) and see if there is a failure and if it looks as expected:

```javascript
// was: t.assertThrow("some error", ...);
t.assertThrow("some error", function () {
    // ...
});
// => Error: Expected to equal some error,
//    but got: Expected all tests to run

// was: t.assertNotThrow(...);
t.assertThrow("some error", function () {
    // ...
});
// => Error: Expected to throw an error,
//    but nothing was thrown
```

And it fails as expected, which means that our refactored tests still work as they should.

> We have just applied a neat technique here: whenever we do a major refactoring in tests, we need to make sure they are still functioning correctly. For that, we break every single one of them (by changing the assertion or breaking the production code). Then we see if they fail as we expect them to. When they don't, we know that refactoring didn't quite work.

### Fixing test suites to run all tests

Now we can go back to the `RunTestSuiteTest` and see if it works as expected without that test. And it does: `Error: Expected all tests to run`. To fix that we need to provide a process spy in every inner call to `runTestSuite`. For that we will first extract `{reporter: reporter}` as a common variable of the test suite:

```javascript
var options = {reporter: reporter};

// ...
// all the inner calls to "runTestSuite":
runTestSuite(function(t) {
    // ...
}, options);
```

And to make the error go away, we now can create a process spy and provide it through options:

```javascript
var process = new ProcessSpy(t);
var options = {
    reporter: reporter,
    process: process
};
```

If we run tests now, they all pass. And we can see that they all execute. Now we just need to double-check that all tests, that have inner calls to `runTestSuite` have `verifyAllTestsRun` option enabled. The only other test suite is the `FailureTest`. Adding the option does not produce a failure because this test suite already uses process spy in all inner calls to `runTestSuite`.

## Conclusion

Today we learned that it is tricky to work with `process.exit` or any function that can exit our program in the middle of the test. Such functions need to be mocked out completely inside of the tests. Also, we learned that it is possible to make sure we don't forget to do that. That is quite important because, if we do forget, everything runs smoothly, and we don't know that we made a mistake.

There is still a lot to go through. In a few next episodes we will:

- Report OK and FAIL for each test;
- Output carefully formatted failures to the STDERR;
- Enable our testing framework to run multiple test suite files at once;
- Enable our testing framework to run in a browser (it is javascript after all).

See you reading the next exciting article of the series: "Formatting the Output"!

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on Twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don’t hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).