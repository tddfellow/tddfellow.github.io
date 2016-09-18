---
layout: post
title: "Build Your Own Testing Framework. Part 4"
description: "As you might have noticed, currently, our testing framework only outputs failures and nothing else. It is impossible to know if it actually runs any tests when they all pass because there is no output. Today we will implement a simple reporter for our testing framework."
date: 2016-09-17 10:00:32 +0200
comments: true
published: false
categories:
- TDD
- test-driven-development
- testing
- xUnit
- build-your-own-testing-framework
- javascript
- duck-typing
- test-doubles
---

Welcome back to the new issue of "Build Your Own Testing Framework" series! As you might have noticed, currently, our testing framework only outputs failures and nothing else. It is impossible to know if it actually runs any tests when they all pass because there is no output. Today we will implement a simple reporter for our testing framework. It will report the name of the test suite and names of the tests that are being executed, for example:

```
SpyTest
	testIsNotCalledInitially
	testAssertNotCalledFailsWhenWasCalled
	testIsCalledAfterBeingCalled
	testAssertCalledFailsWhenWasNotCalled
```

This article is the fourth one of the series "Build Your Own Testing Framework", so make sure to stick around for next parts! All articles of these series can be found [here](/blog/categories/build-your-own-testing-framework/).

Shall we get started?

## Render the name of the test suite

So where should the name of the test suite come from? Probably it should be a test suite class name. Currently all of them are anonymous classes and therefore don't have a name:

```javascript
runTestSuite(function () {
  //         ^          ^
  //       - no name here -
  // ...
});
```

We would like all test suites to have that name, for example:

```javascript
runTestSuite(function SpyTest() {
  //                 ^       ^
  //            - here is the name -
  // ...
});
```

We should write a test for this case:

1. Create a test suite with the name
2. Run the test suite with function `runTestSuite`
3. Assert that the test suite name is reported

Let's try to write a test in a `RunTestSuiteTest.js` test suite for that:

```javascript
this.testItOutputsNameOfTheTest = function () {
  runTestSuite(function TestSuiteName(t) {});

  // TODO: assert that the test suite name is reported
};
```

Now it is problematic: how are we going to assert that something is reported? Should we replace `console.log(message)` or `process.stdout.write(message)` with our own implementation, so that we can test it?:

```javascript
var logged = "";
var oldConsoleLog = console.log;

console.log = function (message) {
  logged = logged + message + "\n";
};
```

And then we should be able to assert with: `t.assertTrue(logged.indexOf("TestSuiteName") >= 0)`. Finally we will need to restore the old `console.log` function:

```javascript
this.testItOutputsNameOfTheTest = function () {
  var logged = "";
  var oldConsoleLog = console.log;

  console.log = function (message) {
    logged = logged + message + "\n";
  };

  runTestSuite(function TestSuiteName(t) {});

  t.assertTrue(logged.indexOf("TestSuiteName" >= 0));

  console.log = oldConsoleLog;
};
```

While this code works, it has multitude of problems:

- If the test fail then `oldConsoleLog` function is not restored;
- It has too much setup (which we could extract as a function);
- It has teardown (which would be nice to avoid if we could);
- It is hard to read, because from 8 lines of code only 2 are delivering the core intent;
- And it is testing how exactly test suite name is being reported, which is basically a View-like concern.

And fixing the last problem will actually fix everything else because this problem causes others. We can fix it by introducing some sort of `Reporter` type, that can respond to `reportTestSuite(name)` message:

```javascript
this.testItOutputsNameOfTheTest = function () {
  runTestSuite(function TestSuiteName(t) {
  }, {reporter: reporter});

  t.assertTrue(reporter.hasReportedTestSuite("TestSuiteName"));
  // or even better:
  reporter.assertHasReportedTestSuite("TestSuiteName");
};
```

`reporter` in this case is some sort of test double. And what are they? - Find out here: [Introducing Test Doubles](/blog/2016/09/18/introducing-test-doubles/).

## Implementing the reporter spy

So our `reporter` object in the test seems terribly like a Spy Double to me, let's test-drive it:

```javascript
// test/ReporterSpyTest.js
var runTestSuite = require("../src/TestingFramework");
var ReporterSpy = require("./ReporterSpy");

runTestSuite(function ReporterSpy_BehaviorTest(t) {
  var reporter = new ReporterSpy(t);

  // Let's write our first test:
  this.testAssertHasReportedTestSuite_whenFailing = function () {
    t.assertThrow(
      "Expected test suite 'HelloWorld' to be reported",
      function () {
        reporter.assertHasReportedTestSuite("HelloWorld");
      }
    );
  };
});

// Error: Cannot find module './ReporterSpy'

// Create file test/ReporterSpy.js
```

Now we are getting the following error:

```
//     var reporter = new ReporterSpy(t);
//                    ^
//
// TypeError: ReporterSpy is not a function
```

We need to create ReporterSpy object now:

```javascript
module.exports = function ReporterSpy(assertions) {

};
```

Now we are getting:

```
// Error: Expected to equal
//   Expected test suite 'HelloWorld' to be reported,
// but got:
//   reporter.assertHasReportedTestSuite is not a function
```

Now we need to create a function `assertHasReportedTestSuite(name)` for out `ReporterSpy`:

```javascript
this.assertHasReportedTestSuite = function (expectedName) {
  assertions.assertTrue(
    false,
    "Expected test suite 'HelloWorld' to be reported"
  );
};
```

Next we need to make sure, that `expectedName` is actually present in the error message by triangulating with different name:

```javascript
this.testAssertHasReportedTestSuite_whenFailing_withOtherName = function () {
  t.assertThrow("Expected test suite 'OtherTestSuite' to be reported", function () {
    reporter.assertHasReportedTestSuite("OtherTestSuite");
  });
};

// Error: Expected to equal
//   Expected test suite 'OtherTestSuite' to be reported,
// but got:
//   Expected test suite 'HelloWorld' to be reported

// And we need to change the respective string:
"Expected test suite '" + expectedName + "' to be reported"
```

Then we need to make sure that we do succeed when the message is received:

```javascript
this.testAssertHasReportedTestSuite_whenSucceeding = function () {
  t.assertNotThrow(function () {
    reporter.reportTestSuite("HelloWorld");
    reporter.assertHasReportedTestSuite("HelloWorld");
  });
};

// Error:
//   Expected not to throw error,
// but thrown
//   'reporter.reportTestSuite is not a function'

// So we need to define this function in ReporterSpy:
this.reportTestSuite = function (name) {

};

// Error:
//   Expected not to throw error,
// but thrown
//   'Expected test suite 'HelloWorld' to be reported'

// Now we need to provide the simplest implementation we can,
// we can do that by introducing the boolean variable:

module.exports = function ReporterSpy(assertions) {
  // initially nothing is reported
  var hasReported = false;

  this.assertHasReportedTestSuite = function (expectedName) {
    assertions.assertTrue(
      // we should fail only when nothing was reported
      hasReported,
      "Expected test suite '" + expectedName + "' to be reported"
    );
  };

  this.reportTestSuite = function (name) {
    // and we mark it as reported when we do receive the message
    hasReported = true;
  };
};
```

And all our tests pass. Now, when the wrong name is getting reported we should still fail:

```javascript
this.testAssertHasReportedTestSuite_whenReporting_andFailing = function () {
  t.assertThrow("Expected test suite 'HelloWorld' to be reported", function () {
    reporter.reportTestSuite("OtherTestSuite");
    reporter.assertHasReportedTestSuite("HelloWorld");
  });
};

// Error: Expected to throw an error,
// but nothing was thrown

// Now we need to actually store the name of reported test suite:

module.exports = function ReporterSpy(assertions) {
  // initially, we didn't receive any reports
  var testSuiteName = null;

  this.assertHasReportedTestSuite = function (expectedName) {
    assertions.assertTrue(
      // we fail only if received testSuiteName is not right
      testSuiteName === "HelloWorld",
      "Expected test suite '" + expectedName + "' to be reported"
    );
  };

  this.reportTestSuite = function (name) {
    // and we need to store the reported name
    testSuiteName = name;
  };
};
```

And all tests pass again. Although, we should notice this weird condition:

```javascript
testSuiteName === "HelloWorld"
```

Looks like our current production code is not generic enough, it will work well only with the `expectedName` equal to `"HelloWorld"`. Let's fix that by triangulating over this parameter:

```javascript
this.testAssertHasReportedTestSuite_whenReporting_andFailingWithDifferentName = function () {
  t.assertThrow("Expected test suite 'OtherTestSuite' to be reported", function () {
    reporter.reportTestSuite("HelloWorld");
    reporter.assertHasReportedTestSuite("OtherTestSuite");
  });
};

// Error: Expected to throw an error,
// but nothing was thrown

// And we should fix it by actually using the `expectedName`:

assertions.assertTrue(
  testSuiteName === expectedName,
  //               ^ fixed here ^
  "Expected test suite '" + expectedName + "' to be reported"
);
```

And all the tests pass. Now we can get back to our failing test for the `runTestSuite`:

## Implementing rendering of the name of the test suite

```javascript
this.testItOutputsNameOfTheTest = function () {
  runTestSuite(function TestSuiteName(t) {
  }, {reporter: reporter});

  reporter.assertHasReportedTestSuite("TestSuiteName");
};
```

To implement this, first we will need to accept `options` parameter with sane defaults:

```javascript
function runTestSuite(testSuiteConstructor, options) {
  options = options || {};
  var reporter = options.reporter || new SimpleReporter();

  // ...
}

// We have to implement this, otherwise our test suite will fail
function SimpleReporter() {
    this.reportTestSuite = function (name) {
        process.stdout.write("\n" + name + "\n");
    };
}
```

After making the failing test pass and triangulating over the name of the test suite:

```javascript
function runTestSuite(testSuiteConstructor, options) {
  options = options || {};
  var reporter = options.reporter || new SimpleReporter();

  reporter.reportTestSuite(testSuiteConstructor.name)

  // ...
}
```

And all tests pass now. Unfortunately, this is the ouput that we see now:

```







```

Yeah, empty lines. This is because `(function () {}).name` is equal to `""`. We need to give proper names to all our anonymous constructors for the test suites:

```javascript
runTestSuite(function RunTestSuiteTest(t) { ... });
runTestSuite(function AssertEqualTest(t) { ... });
// .. and so on ..
```

And now we should see the correct output:

```
AssertEqualTest

AssertNotEqualTest

AssertNotThrowTest

AssertThrowTest

AssertTrueTest

FizzBuzzKataTest

.. and so on ..
```

Great, now we would like to render the name of the executed test:

## Render the name of the executed test

```javascript
this.testItOutputsNameOfTheTest = function () {
  runTestSuite(function TestSuiteName(t) {
    this.testSomeTestName = function () {};
    this.testSomeOtherTestName = function () {};
  }, {reporter: reporter});

  reporter.assertHasReportedTestSuite("TestSuiteName");
  reporter.assertHasReportedTest("testSomeTestName");
  reporter.assertHasReportedTest("testSomeOtherTestName");
};
```

Of course this fails, because we need to implement `assertHasReportedTest(name)` now for our `ReporterSpy`. Let's test-drive it:

```javascript
// test/ReporterSpyTest.js
this.testAssertHasReportedTest_whenFailing = function () {
  t.assertThrow("Expected test 'testName' to be reported", function () {
    reporter.assertHasReportedTest("testName");
  });
};

// Error: Expected to equal
//   Expected test 'testName' to be reported,
// but got:
//   reporter.assertHasReportedTest is not a function

// We need to define assertHasReportedTest(name) method:
this.assertHasReportedTest = function (expectedName) {

};

// Error: Expected to throw an error,
// but nothing was thrown

// We need to make it throw the expected error:
this.assertHasReportedTest = function (expectedName) {
  assertions.assertTrue(
    false,
    "Expected test 'testName' to be reported"
  );
};

// And the test passes. Message hard-codes `testName` -
// we should triangulate over it:

this.testAssertHasReportedTest_whenFailing_withDifferentName = function () {
  t.assertThrow("Expected test 'testDifferentName' to be reported", function () {
    reporter.assertHasReportedTest("testDifferentName");
  });
};

// Error: Expected to equal
//   Expected test 'testDifferentName' to be reported,
// but got:
//   Expected test 'testName' to be reported

// And to fix it:
"Expected test '" + expectedName + "' to be reported"

// Next test will force us to implement simple reportTest function:
this.testAssertHasReportedTest_whenSucceeding = function () {
  t.assertNotThrow(function () {
    reporter.reportTest("testName");
    reporter.assertHasReportedTest("testName");
  });
};

// Error: reporter.reportTest is not a function

// After fixing this and triangulating a bit, we get:

module.exports = function ReporterSpy(assertions) {
  var testName = null;
  // ...
  this.assertHasReportedTest = function (expectedName) {
    assertions.assertTrue(
      testName === expectedName,
      "Expected test '" + expectedName + "' to be reported"
    );
  };
  this.reportTest = function (name) {
    testName = name;
  };
}

// Finally we need ability to report multiple tests:

this.testAssertHasReportedTest_whenSucceeding_withMultipleReports = function () {
  t.assertNotThrow(function () {
    reporter.reportTest("testName");
    reporter.reportTest("testOtherName");
    reporter.assertHasReportedTest("testName");
  });
};

// Error: Expected not to throw error,
// but thrown 'Expected test 'testName' to be reported'

// And to implement this:
module.exports = function ReporterSpy(assertions) {
  // we will store all reported names,
  // initially no names are reported
  var testNames = [];
  // ...
  this.assertHasReportedTest = function (expectedName) {
    assertions.assertTrue(
      // check if expectedName was reported
      testNames.indexOf(expectedName) >= 0,
      "Expected test '" + expectedName + "' to be reported"
    );
  };
  this.reportTest = function (name) {
    // store the reported test name
    testNames.push(name);
  };
}
```

Unfortunately, this does not pass our tests, because this test fails now:

```javascript
this.testAssertHasReportedTest_whenReporting_andFailing = function () {
  t.assertThrow("Expected test 'testName' to be reported", function () {
    reporter.reportTest("testOtherName");
    reporter.assertHasReportedTest("testName");
  });
};
```

After an investigation, it becomes clear, that this happens because we can not re-use `reporter` variable defined at the higher level, since all tests share the same `testSuite` object at the moment. We will have to move the creation of the `reporter` variable inside of each test:

```javascript
this.testAssertHasReportedTest_whenReporting_andFailing = function () {
  var reporter = new ReporterSpy(t);
  // ...
};

this.testAssertHasReportedTest_whenReporting_andFailing_withOtherName = function () {
  var reporter = new ReporterSpy(t);
  // ...
};

// .. and so on ..
```

And this makes all our tests pass.

## Stateless tests

This is quite noticeable problem, that our users can be frustrated with, so we probably should make it easy on them and allow such variables to be fresh for every test. This can be achieved quite easy if we were to create a new `testSuite` for each test. Let's write a simple test to show the problem:

```javascript
// test/StatelessTest.js
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function StatelessTest(t) {
  var answer = 41;

  this.testItCanMutateVariable_andImmediatelyUseNewValue = function () {
    answer++;
    t.assertEqual(42, answer);
  };

  this.testItCanMutateVariableAgain_andGetTheSameResult = function () {
    answer++;
    t.assertEqual(42, answer);
  };
  // this fails as expected:
  // Error: Expected to equal 42, but got: 43
});
```

And now let's implement it by creating the `testSuite` for every test:

```javascript
function runTestSuite(testSuiteConstructor, options) {
  options = options || {};
  var reporter = options.reporter || new SimpleReporter();

	reporter.reportTestSuite(testSuiteConstructor.name);

	var testSuitePrototype = createTestSuite(testSuiteConstructor);
	// ^ we change this from `testSuite` to `testSuitePrototype`  ^

  for (var testName in testSuitePrototype) {
    if (testName.match(/^test/)) {
      var testSuite = createTestSuite(testSuiteConstructor);
			// ^   and we create our testSuite every time here   ^
      testSuite[testName]();
	// ^  and run test on it ^
    }
  }
}

function createTestSuite(testSuiteConstructor) {
    return new testSuiteConstructor(assertions);
}
```

After doing this, we can move `var reporter = new ReporterSpy(t);` to the top level of the `ReporterSpyTest` suite again. And all the tests pass.

## Implementation of the rendering of the test name

Finally, we need to make sure that the test suite, that we have written before will pass:

```javascript
this.testItOutputsNameOfTheTest = function () {
	runTestSuite(function TestSuiteName(t) {
		this.testSomeTestName = function () {};
		this.testSomeOtherTestName = function () {};
	}, {reporter: reporter});

	reporter.assertHasReportedTestSuite("TestSuiteName");
	reporter.assertHasReportedTest("testSomeTestName");
	reporter.assertHasReportedTest("testSomeOtherTestName");
};
```

As expected it fails with `Error: Expected test 'testSomeTestName' to be reported`. After fixing it and applying triangulation once, we would end up with the following implementation:

```javascript
// src/TestingFramework.js in runTestSuite function:
for (var testName in testSuitePrototype) {
	if (testName.match(/^test/)) {

		reporter.reportTest(testName);
// ^  here is our implementation  ^

		var testSuite = createTestSuite(testSuiteConstructor);
		testSuite[testName]();
	}
}

function SimpleReporter() {
	// ...
	// and we should not forget to implement it for real reporter
  this.reportTest = function (name) {
    process.stdout.write("\t" + name + "\n");
  };
}
```

Now, it seems that both `ReporterSpy` and `SimpleReporter` are implementing the same Duck type - `Reporter`. What Duck Type is? - find out here: [Meet Duck Type](/blog/2016/09/18/meet-duck-type/).

## Contract testing all Reporter duck types

So we should test all our ducks that their public API don't get out of sync:

```javascript
var TestingFramework = require("../src/TestingFramework");
var runTestSuite = TestingFramework;
var SimpleReporter = TestingFramework.SimpleReporter;

var ReporterSpy = require("./ReporterSpy");

const IMPLEMENTATIONS = [
    SimpleReporter,
    ReporterSpy
];

IMPLEMENTATIONS.forEach(function (ReporterImplementation) {
  runTestSuite(function (t) {
    var reporter = new ReporterImplementation();

    this.testDefines_reportTestSuite = function () {
      var reportTestSuite = reporter.reportTestSuite;
      t.assertEqual("function", typeof(reportTestSuite));
      t.assertEqual(1, reportTestSuite.length);
    };

    this.testDefines_reportTest = function () {
      var reportTest = reporter.reportTest;
      t.assertEqual("function", typeof(reportTest));
      t.assertEqual(1, reportTest.length);
    }
  });
});
```

All the tests pass. Unfortunately, the output regarding this test suite looks weird:

```

	testDefines_reportTestSuite
	testDefines_reportTest


	testDefines_reportTestSuite
	testDefines_reportTest
```

The test suite name is empty. I think we need an ability to define a custom and dynamic test suite name:

## Custom name for the test suite

We can achieve this by allowing any test suite to define special hook method, that will return its custom name, like `testSuite.getTestSuiteName()`. Let's write a test for this:

```javascript
this.testItCanHaveCustomNameOfTheTestSuite = function () {
  runTestSuite(function (t) {
    this.getTestSuiteName = function () {
      return "CustomNameOfTheTestSuite";
    };
  }, {reporter: reporter});

  reporter.assertHasReportedTestSuite("CustomNameOfTheTestSuite");
};
```

After implementing it and triangulating over the name once the code looks like this:

```javascript
function runTestSuite(testSuiteConstructor, options) {
  options = options || {};
  var reporter = options.reporter || new SimpleReporter();

  var testSuitePrototype = createTestSuite(testSuiteConstructor);

  reporter.reportTestSuite(
    getTestSuiteName(testSuiteConstructor, testSuitePrototype)
// ^ this is the function that we introduced here to make it pass ^
  );

	for (var testName in testSuitePrototype) { ... }
}

function getTestSuiteName(testSuiteConstructor, testSuitePrototype) {
    if (typeof(testSuitePrototype.getTestSuiteName) !== "function") {
        return testSuiteConstructor.name;
    }

    return testSuitePrototype.getTestSuiteName();
}
```

Now, if we were to use this feature in our duck type tests:

```javascript
IMPLEMENTATIONS.forEach(function (ReporterImplementation) {
  runTestSuite(function (t) {
    this.getTestSuiteName = function () {
      return ReporterImplementation.name + "_ReporterTest";
    };

	// ...
});
```

Then we are getting the proper output:

```
SimpleReporter_ReporterTest
	testDefines_reportTestSuite
	testDefines_reportTest

ReporterSpy_ReporterTest
	testDefines_reportTestSuite
	testDefines_reportTest
```

## Bottom Line

I think we are done with implementing our first simple reporter. Now we can see that the tests are actually executing and passing. The code can be found here: https://github.com/waterlink/BuildYourOwnTestingFrameworkPart4

There is still a lot to go through. In a few next episodes we will:

- Make sure that first failure does not cause test suite to stop running;
- Make sure the exit code is right;
- Report OK and FAIL;
- Output carefully formatted failures to the STDERR.

Stay tuned!

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
