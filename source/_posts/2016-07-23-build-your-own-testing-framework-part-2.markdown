---
layout: post
title: "Build Your Own Testing Framework. Part 2"
date: 2016-07-23 14:52:42 +0200
comments: true
categories:
- TDD
- test-driven-development
- testing
- xUnit
- build-your-own-testing-framework
- javascript
---

Today we are going to unit-test existing functionality of our own testing framework, that we have test-driven with `FizzBuzzKata` in the previous part.

This needs to be done, since currently only happy paths are implicitly tested via `FizzBuzzKata` test suite, i.e.: when all the tests pass. The following unhappy paths are not tested at the moment:

- when `assertTrue` fails, it throws an error,
- when `assertEqual` fails, it throws an error,
- when the test fails, it renders the error and fails the whole test suite run, i.e.: non-zero exit code.

This article is the second one of the series "Build Your Own Testing Framework", so make sure to stick around for next parts! All articles of these series can be found here: http://www.tddfellow.com/blog/categories/build-your-own-testing-framework/.

Shall we get the ball rolling?

## Testing `assertTrue(condition, message)`

Let's create a test suite for `assertTrue` with the first test for the case, when `assertTrue` succeeds:

```javascript
// test/AssertTrueTest.js
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function (t) {
    this.testSuccess = function () {
        t.assertTrue(true);
    };
});
```

If we run this test suite, it should not fail:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Let's check if that test actually is testing anything by trying to break the implementation of `assertTrue`:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (condition) {    // <- this condition was inverted
        throw new Error(...);
    }
},
```

And if we run it, we get the expected error:

```
/usr/local/bin/node AssertTrueTest.js
/path/to/project/src/TestingFramework.js:4
            throw new Error(message || "Expected to be true, but got false");
            ^

Error: Expected to be true, but got false
    .. stack trace ..

Process finished with exit code 1
```

OK, it fails as expected, now we should undo our breaking change in the implementation and run the test suite again and it should pass:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (!condition) {    // <- the breaking change here have been undone
        throw new Error(...);
    }
},
```

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Now, let's write a new test for the case, when `assertTrue` fails:

```javascript
// test/AssertTrueTest.js

this.testFailure = function () {
    try {
        t.assertTrue(false);
    } catch (error) {
        t.assertEqual("Expected to be true, but got false", error.message);
    }
};
```

And if we run the test suite, it passes. If I was confident in the previous test, that I know why it didn't fail, here I feel a bit uncomfortable about writing these 5 lines of test code and never seeing them fail. So let's break the code once again!

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (!condition) {
        throw new Error(message || "oops");   // <- we have changed error message here
    }
},
```

And if we run tests, we get an expected failure:

```
/usr/local/bin/node AssertTrueTest.js
/path/to/project/src/TestingFramework.js:4
            throw new Error(message || "oops");
            ^

Error: Expected to equal Expected to be true, but got false, but got: oops
    .. stack trace ..

Process finished with exit code 1
```

And if we undo our breaking change in the implementation the test should pass:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (!condition) {
        throw new Error(message || "Expected to be true, but got false");
        //                         ^ we have restored correct message ^
    }
},
```

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

And the last test for `assertTrue` to test the custom failure message:

```javascript
// test/AssertTrueTest.js

this.testCustomFailureMessage = function () {
    try {
        t.assertTrue(false, "it is not true!");
    } catch (error) {
        t.assertEqual("it is not true!", error.message);
    }
};
```

If we run the test suite, it passes:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Even that I am confident enough, that this `try { ... } catch (..) { ... }` construction does the right thing, let's be diligent about it and break the implementation of that functionality and see this test fail:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (!condition) {
        throw new Error("Expected to be true, but got false");
        //              ^ 'message || ' was removed here   ^
    }
},
```

If we run the test suite:

```
/usr/local/bin/node AssertTrueTest.js
/path/to/project/src/TestingFramework.js:4
            throw new Error("Expected to be true, but got false");
            ^

Error: Expected to be true, but got false
    .. stack trace ..

Process finished with exit code 1
```

It fails, but this change breaks `assertEqual` too, so that we don't see any meaningful error message now. We can figure out if that is an expected failure or not by inspecting the stack trace:

```
Error: Expected to be true, but got false
    at Object.assertions.assertTrue (/path/to/project/src/TestingFramework.js:4:19)
    at Object.assertions.assertEqual (/path/to/project/src/TestingFramework.js:9:14)
    at testCustomFailureMessage (/path/to/project/test/AssertTrueTest.js:20:15)
    at runTestSuite (/path/to/project/src/TestingFramework.js:21:32)
    at Object.<anonymous> (/path/to/project/test/AssertTrueTest.js:3:1)
    at Module._compile (module.js:409:26)
    at Object.Module._extensions..js (module.js:416:10)
    at Module.load (module.js:343:32)
    at Function.Module._load (module.js:300:12)
    at Function.Module.runMain (module.js:441:10)
```

and precisely this line is the most interesting one:

```
at testCustomFailureMessage (/path/to/project/test/AssertTrueTest.js:20:15)
```

If we open the code at this stack trace frame, we will see:

```javascript
t.assertEqual("it is not true!", error.message);
```

Great, this is exactly what we expected. Undoing the breaking change and running the test suite again:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    if (!condition) {
        throw new Error(message || "Expected to be true, but got false");
        //              ^ 'message || ' was inserted here again       ^
    }
},
```

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Interestingly enough, I know how to break the code now and all the tests will pass:

First, let's extract local variable in `assertTrue`:

```javascript
// src/TestingFramework.js

assertTrue: function (condition, message) {
    var errorMessage = message || "Expected to be true, but got false";

    if (!condition) {
        throw new Error(errorMessage);
    }
},
```

And that still passes its tests:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Now let's expand `message || ` into the proper `if` statement:

```javascript
var errorMessage = "Expected to be true, but got false";

if (message) {
    errorMessage = message;
}
```

And that still passes its tests:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

Now, what if I were to replace `errorMessage = message;` with `errorMessage = "it is not true!";` inside of the `if` statement:

```javascript
if (message) {
    errorMessage = "it is not true!";
}
```

The test still passes!

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

This is rather interesting. It seems, that we will have to make sure, that `message` parameter actually gets passed to `new Error(...)`. We can do that by applying the Triangulation technique focusing on `message` parameter:

```javascript
// test/AssertTrueTest.js

this.testCustomFailureMessage_withOtherMessage = function () {
    try {
        t.assertTrue(false, "should be true");
    } catch (error) {
        t.assertEqual("should be true", error.message);
    }
};
```

And if we run the test suite, we get our expected failure:

```
/usr/local/bin/node AssertTrueTest.js
/path/to/project/src/TestingFramework.js:10
            throw new Error(errorMessage);
            ^

Error: it is not true!
    ...
    at testCustomFailureMessage_withOtherMessage (/path/to/project/test/AssertTrueTest.js:28:15)
    ...

Process finished with exit code 1
```

And if we look at the stack trace frame, it points us to the correct line of code:

```javascript
t.assertEqual("should be true", error.message);
```

And we can now fix the test by fixing the implementation:

```javascript
// src/TestingFramework.js

// inside of `assertTrue` function:
if (message) {
    errorMessage = message;   // <- here we actually have to use 'message' now
}
```

And when we run the test suite, it passes:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

I think we have all required tests for `assertTrue` now. This is great! And have you spotted the duplication already?

## Refactoring `AssertTrueTest`

The duplication that is present here is our `try { ... } catch (..) { ... }` construct. Let's make it a completely duplicate code by extracting couple of local variables:

```javascript
// first, let's extract the Action Under the Test variable:
var action = function () { t.assertTrue(false); };

try {
    action();
} catch (error) {
    t.assertEqual("Expected to be true, but got false", error.message);
}

// next, let's extract the Expected Error Message:
var expectedMessage = "Expected to be true, but got false";

try {
    action();
} catch (error) {
    t.assertEqual(expectedMessage, error.message);
}
```

If we apply the same refactoring for all tests in the `AssertTrueTest` suite, we will see the duplication:

```javascript
try {
    action();
} catch (error) {
    t.assertEqual(expectedMessage, error.message);
}
```

Let's give that operation a name and extract is as a function: `assertThrow(expectedMessage, action)`:

```javascript
function assertThrow(expectedMessage, action) {
    try {
        action();
    } catch (error) {
        t.assertEqual(expectedMessage, error.message);
    }
}
```

Use this function in all tests and inline all the extracted local variables:

```javascript
this.testFailure = function () {
    assertThrow("Expected to be true, but got false", function () {
        t.assertTrue(false);
    });
};

this.testCustomFailureMessage = function () {
    assertThrow("it is not true!", function () {
        t.assertTrue(false, "it is not true!");
    });
};

this.testCustomFailureMessage_withOtherMessage = function () {
    assertThrow("should be true", function () {
        t.assertTrue(false, "should be true");
    });
};
```

Function `assertThrow` seems to be useful for any test, not just `AssertTrueTest` suite. Let's move it to the `assertions` object. We will be doing that by using Parallel Change technique:

1. Create new functionality first;
2. Step by step migrate all calls to use new functionality instead of old one;
3. Once old functionality is not used, remove it.

Advantage of that method, is that it consists of very small steps, that can be executed with confidence and each such small step never leaves the user in red state (failing tests or compile/parsing errors).

Let's see how this one can be applied here:

`1. Create new functionality first` - we can do it by copying the `assertThrow` function to `assertions` object:

```javascript
// src/TestingFramework.js

var assertions = {
    // ...
    assertThrow: function (expectedMessage, action) {
        try {
            action();
        } catch (error) {
            // `t` needs to be changed to `this` here
            this.assertEqual(expectedMessage, error.message);
        }
    }
};
```

The tests should still pass:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

`2. Step by step migrate all calls to use new functionality instead of old one` - we do it by calling `assertThrow` on `t` (our assertions object) in the test suite. Since we still haven't removed the old `assertThrow` function, we can do that one function call at the time and the tests will always be green:

```javascript
this.testFailure = function () {
    t.assertThrow("Expected to be true, but got false", function () {
// ^ 't.' added here ^
        t.assertTrue(false);
    });
};

// run tests and they still pass.

this.testCustomFailureMessage = function () {
    t.assertThrow("it is not true!", function () {
// ^ 't.' added here ^
        t.assertTrue(false, "it is not true!");
    });
};

// run tests and they still pass.

this.testCustomFailureMessage_withOtherMessage = function () {
    t.assertThrow("should be true", function () {
// ^ 't.' added here ^
        t.assertTrue(false, "should be true");
    });
};
```

`3. Once old functionality is not used, remove it.` - now we can remove our `assertThrow` function defined inside of the `AssertTrueTest` suite and run tests:

```
/usr/local/bin/node AssertTrueTest.js

Process finished with exit code 0
```

And they pass. Let's see the complete `AssertTrueTest` suite again:

```javascript
// test/AssertTrueTest.js
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function (t) {
    this.testSuccess = function () {
        t.assertTrue(true);
    };

    this.testFailure = function () {
        t.assertThrow("Expected to be true, but got false", function () {
            t.assertTrue(false);
        });
    };

    this.testCustomFailureMessage = function () {
        t.assertThrow("it is not true!", function () {
            t.assertTrue(false, "it is not true!");
        });
    };

    this.testCustomFailureMessage_withOtherMessage = function () {
        t.assertThrow("should be true", function () {
            t.assertTrue(false, "should be true");
        });
    };
});
```

The only problem here, is that we are relying on the untested `assertThrow` assertion here. Let's unit-test it.

## Testing `assertThrow(expectedMessage, action)`

Let's create a new test suite with first test when `assertThrow` succeeds:

```javascript
// test/AssertThrowTest.js
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function (t) {
    this.testSuccess = function () {
        assertThrow("an error message", function () {
            throw new Error("an error message");
        });
    };
});
```

And the test should pass:

```javascript
var runTestSuite = require("../src/TestingFramework");

runTestSuite(function (t) {
    this.testSuccess = function () {
        t.assertThrow("an error message", function () {
            throw new Error("an error message");
        });
    };
});
```

The code can be broken without test failure by not using `expectedMessage` parameter:

```javascript
// src/TestingFramework.js

assertThrow: function (expectedMessage, action) {
    try {
        action();
    } catch (error) {
        this.assertEqual("an error message", error.message);
        // ^ here 'expectedMessage' was changed to constant ^
    }
}
```

Let's triangulate the code to make sure `expectedMessage` is used correctly by adding a new test:

```javascript
// test/AssertThrowTest.js

this.testSuccess_withDifferentExpectedMessage = function () {
    t.assertThrow("a different error message", function () {
        throw new Error("a different error message");
    });
};
```

And that test fails as expected:

```
/usr/local/bin/node AssertThrowTest.js
/path/to/project/src/TestingFramework.js:10
            throw new Error(errorMessage);
            ^

Error: Expected to equal an error message, but got: a different error message

Process finished with exit code 1
```

Undoing the breaking change (Mutation) will make the test pass:

```
/usr/local/bin/node AssertThrowTest.js

Process finished with exit code 0
```

Next test is to make sure, that `assertThrow` is actually comparing actual and expected error messages correctly:

```javascript
// test/AssertThrowTest.js

this.testFailure = function () {
    t.assertThrow("Expected to equal an error message, but got: a different error message", function () {
        t.assertThrow("an error message", function () {
            throw new Error("a different error message");
        });
    });
};
```

And it passes. The last test, that `assertThrow` needs is the case, when `action()` is not throwing any error. In that case `assertThrow` should fail:

```javascript
// test/AssertThrowTest.js

this.testFailure_whenActionDoesNotThrow = function () {
    t.assertThrow("Expected to throw an error, but nothing was thrown", function () {
        t.assertThrow("an error message", function () {
            // does nothing
        });
    });
};
```

Oh, and that test is passing! We clearly don't have that functionality yet. We have to add another test without usage of outer `t.assertThrow` to make sure that we get a test failure:

```javascript
this.testThrows_whenActionDoesNotThrow = function () {
    var hasThrown = false;

    try {
        t.assertThrow("an error message", function () {
            // does nothing
        });
    } catch (error) {
        hasThrown = true;
    }

    t.assertTrue(hasThrown, "it should have thrown");
};
```

And if we run our tests we get the expected failure:

```
/usr/local/bin/node AssertThrowTest.js
/path/to/project/src/TestingFramework.js:10
            throw new Error(errorMessage);
            ^

Error: it should have thrown

Process finished with exit code 1
```

We can fix that by verifying, that `catch` block is executed in the `assertThrow` function:

```javascript
// src/TestingFramework.js

assertThrow: function (expectedMessage, action) {
    var hasThrown = false;  // <- we initialize hasThrown here

    try {
        action();
    } catch (error) {
        hasThrown = true;   // <- and we set it to true in the catch block
        this.assertEqual(expectedMessage, error.message);
    }

    this.assertTrue(hasThrown);  // <- and we check that it is true
}
```

And now if we run the test suite, the original test that we were trying to write fails as expected:

```
/usr/local/bin/node AssertThrowTest.js
/path/to/project/src/TestingFramework.js:10
            throw new Error(errorMessage);
            ^

Error: Expected to equal Expected to throw an error, but nothing was thrown, but got: Expected to be true, but got false

Process finished with exit code 1
```

Now we need to fix the custom message provided to `assertTrue` call inside of `assertThrow`:

```javascript
// src/TestingFramework.js

// inside of assertThrow:
this.assertTrue(hasThrown, "Expected to throw an error, but nothing was thrown");
```

And the test pass:

```
/usr/local/bin/node AssertThrowTest.js

Process finished with exit code 0
```

Great, I think we are done with testing the `assertThrow` function. Last one for today is `assertEqual`:

## Testing `assertEqual(expected, actual)`








## Next time

Additionally, today we are going to add the following two features to our testing framework:

    do not stop execution on first failure and run all tests, and
    report amount of total tests and amount of tests that have failed.
