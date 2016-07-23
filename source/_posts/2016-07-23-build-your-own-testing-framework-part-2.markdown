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

## Testing `assertTrue`

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










## Next time

Additionally, today we are going to add the following two features to our testing framework:

    do not stop execution on first failure and run all tests, and
    report amount of total tests and amount of tests that have failed.
