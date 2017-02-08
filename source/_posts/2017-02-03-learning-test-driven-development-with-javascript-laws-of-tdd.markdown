---
layout: post
title: "Learning Test-Driven Development With Javascript: Laws of TDD"
date: 2017-02-03 22:19:40 +0100
comments: true
categories:
- tdd
- tutorial
- learning-tdd-with-js
- javascript
---

**Level: Beginner**

Today we are going to learn the basic principles behind the Test-Driven Development Discipline. We will learn three rules of TDD. And we will take a look at the example application of these laws.

<!-- more -->

Articles of these series have exercises and going through them would make for more effective learning. Also, if you feel stuck with these exercises, fell free to shoot me an email or get in touch on Twitter: @waterlink000.

"Learning Test-Driven Development with Javascript" is a series of articles and you, my dear reader, can shape the content by providing an invaluable feedback. To do that, shoot me an email - oleksii@tddfellow.com.

## Basics of Test-Driven Development

A test is a single program, procedure or function, that executes our system under the test and verifies that it works as expected. The system under the test is any other program, part of the program or library, procedure or function. The system under the test is called SUT for short.

SUT is the code that executes with a purpose of satisfying needs of our end users. Using the term "end user" we mean actual people using our system via the user interface (graphical and non-graphical), and other automated systems using our system via application programming interface (for short, API). A different term for the "end user" is "consumer of the system." Another term for the SUT is "production code" - we are going to use the latter, as it is used in Test-Driven Development much more often.

Test-Driven Development is based on a simple concept of writing production code only when there is a failing test that demands that production code for it to pass.

We call a test failing when there is some error happening when we execute it. Possible failures are the following: there is a syntax error, or code does not compile, or there is a runtime failure during test setup or production code execution, or there is an assertion error. The assertion failure is the particular kind of error that happens during the final phase of the test, where we are verifying the outcomes after running the production code under the test. An assertion error signals that the production code executed successfully, but it produced incorrect results or changed the state of the system in a wrong fashion.

We call a test passing when there are no errors happen when we execute it. We, also, call the failing test "a red test" and we call the passing test "a green test." That is because we can not deliver the code to our customers or consumers when there are "red" tests - think of red traffic light, and we are free to go when all the tests are "green" - think of green traffic light. Most of the testing tools and frameworks format their output to present failing tests in the red color and passing tests in the green color.

Multiple tests aiming to test single SUT or a particular feature of the system are usually called test suite. Depending on the context, test suite could mean that collection of tests all testing the same thing, or it could mean all tests of the entire system. For example, in the sentence "Let's read `User` class' test suite" that phrase means a collection of tests testing class `User`. On the other hand, in the sentence "Let's run the whole test suite and see if we can deploy that right now" that phrase means all tests of the entire system. The latter, sometimes, is called "suite of tests."

When the system behaves in an unexpected way, and the expected behavior was previously defined or present in the code, it is called a "bug." For example, the behavior that is specified by the development team and not implemented correctly considered a bug. The behavior that is defined by the development team and implemented correctly, but now it is not working, considered a bug. And, finally, the behavior that was not defined may or may not be considered as a bug. The latter depends on the produced results - if it harms or brings any value. This phenomenon is called a "bug" for historical reasons: the first bug in computing was a real bug, that stuck in the computer's hardware and was causing short circuits which made the computer misbehave.

In the core of Test-Driven Development there are three steps that we need to follow:

1. We start with a failing test. We consider any error a failure, including compilation and syntax errors. Meaning, that our first test for any part of the production code will fail for the reason that there is no production code to execute yet. For example class, method or function is undefined.
2. Then, we resolve the failure by writing the production code. For example: in the case of missing class, we would create a class; in the event of a missing method, we would create it; and in the instance of a failing assertion error, we would fix the logic to pass the test.
3. We write no more than the simplest production code, which makes our current failing test pass, and, also, still passes all other tests.

We repeat these steps over and over until we finish the implementation of the system under the test. Strictly following these steps will lock us in a very tight loop, where we will be switching between test code and production code all the time: write one or two lines of the test code and write or change one or two lines of the production code, repeat. This cycle is, probably, twenty or thirty seconds long. It, of course, depends on how fast we can run our tests. Ideally, we want our test suite for the current system under the test to run as fast as one clap of hands, or blink of an eye.

This tight cycle gives us following benefits:

- Alerts us to the mistakes we do while coding immediately. Usually, it is enough to do one or two "Undo" commands in our editor to get back to the "green" state of the system - when all our tests are passing (occasionally, except for the last test we have added, as we will undo the incorrect implementation for it).
- Provides the safety net from bugs for the future development thanks to the fact, that no single line of production code is written other than to pass a failing test. Meaning, that whenever production code is broken in any fashion, there is a test that will point it out via failure. When adding new features or modifying the behavior of the existing ones, we might inadvertently introduce a bug. Our suite of tests will detect that bug as soon as we run all our tests, which we would certainly do during development and, especially, before the deployment or release of our systems. This is, basically, a near 100% test coverage. Surely, there are subtle and complex bugs that involve whole sub-systems and 3rd-party systems to come together in some special edge case to appear - rarely, such bugs are not prevented by Test-Driven Development. Of course, we are talking about 1-3 such bugs per year - this is much better than having a bug tracker, that contains thousands of unresolved bugs. Who said it was a good idea to have a database that stores the bugs in our system? - When we apply TDD correctly, and our bug counts are so small, as 1-3 per year, we should treat bugs as "Red Alerts" and concentrate all the effort on shooting them down. Writing a test for these bugs when we already figured them out allows us to programmatically prove, that the system, indeed, does misbehave in a certain edge case, and prevents this bug from reoccurring after we have fixed it.
- Gives the confidence to refactor freely and ruthlessly. Essentially, allows changing the software design of the system on the go without applying the big design upfront (BDUF). Let's take a look at the example. Imagine, we have a system with a perfect design and without any tests. Over time the design will degrade until the code becomes nearly impossible to work with. Now, imagine, we have a system with bad design and close to 100% test coverage. Over time we can improve the design of the application gradually while responding to the changes in the needs of our end users. And we will be faster as the time passes.
- Makes the development team always ready to deploy. While practicing test-driven development, development teams will keep latest implemented features in the version control system for the source code in the state, that is well-known to be "green." This condition of the system is deliverable to the end users with one push of the button. That, also, enables teams to do continuous delivery: as soon as the code is checked-in in the version control system (VCS), it is going to be automatically deployed to production and our end users will get the value out of it sooner.
- Promotes simplicity and alleviates the scope creep. "Oh, we don't need that yet" is the most used phrase in the vocabulary of the typical test-driven development practitioner. Tests that we write specify only the useful functionality that we need to deliver the value to our end users. These tests don't talk about performance optimizations, security concerns, rare edge cases, "re-usable" abstractions (that, usually, are harder to understand and are difficult to re-use), etc. These things need to be implemented only when there is an actual end user value to be achieved, or the system is obviously running slower than it should, or there is an actual security concern. In that case, a very specific test (or tests) will be written to drive out that additional functionality. Since the most software development cost is in the maintenance of the existing functionality leaving out as much functionality (and code) as possible is a good idea.
- Promotes cleaner interfaces. As in test-driven development, we have to write the usage example before we implement (or even design) the API in the production code, it will be optimized for ease and clarity of use and less so with the internal implementation details of that production code. Generally speaking, test-driven production code is easier to use and understand.
- Tests that call the production code serve as usage examples of that code. Essentially, in a test-driven code base, to know how a particular API can be used, it is enough to read and understand its test suite. These tests are the perfect low-level documentation for the APIs in our code base that developers can understand and read best. And that documentation is so precise that it can be executed and verified that it is still in sync with the actual code it is describing. Therefore this documentation is always up-to-date.

The most important of these benefits is the confidence to make any change to the software and know in one minute or two, if that change is good to be delivered to the end user or not, with a simple push of the button. No quality assurance (QA) manual testing cycles are required. Let's take a look at the example of the application of three rules of test-driven development. We will start from a very simple example, so that we don't have to touch more advanced TDD techniques.

### English Numbers Kata

> **Given** an integer number  
> **When** I call the system with that number  
> **Then** I receive a string representation of that number in an English language
> 
> For example:
> 
> - for number `37` I receive `"thirty-seven"`
> - for number `-17451` I receive `"minus seventeen thousand four hundred fifty-one"`

According to the first rule of TDD, we have to start the implementation from the test. In test-driven development it is important to start from the simplest tests, that can be implemented via a small, simple change to production code. For example: with number zero we expect the result to be "zero." When writing the first test for the new functionality we are going to design its API. In our case we are going to come up with the function name and its argument list:

```javascript
describe("toEnglishNumber", function() {
    it("converts 0 to zero", function() {
        // ARRANGE
        var number = 0;
        
        // ACT
        var englishNumber = toEnglishNumber(number);
    });
});
```

As soon as we write `toEnglishNumber(number)` we have designed the function's signature, at least, for the single simplest case. Also, the test suite is failing now, because `toEnglishNumber` is not a function - in fact, it is undefined. That means that we have entered the red stage of test driven development and according to the second rule of TDD we have to switch back to the production code. And according to the third rule, we have to write just enough of it to make the failing test pass. That means writing the simplest and easiest code possible to make it succeed. In this case, we could just return nothing (`null` in javascript):

```javascript
function toEnglishNumber(number) {
    return null;
}
```

This is going to turn our test suite back to the green stage. At this point it is a good idea to look at both test code and production code and see if there are any opportunities for refactoring, such as better names, extracting methods/functions, clarifying variable names, de-duplication, etc. Because we are currently in a green state, we can safely apply any refactoring, automated or manual, and see if it was successful by running the test again. If for some reason, the test is failing after the refactoring, we always have an option to CTRL/CMD+Z back to the green state, back to safety. At this point, we have finished applying one cycle of rules of TDD. Now we need to start over. We go back to our test code. There we extend the existing test to be more specific. Or we add more tests. Since we didn't finish writing the test yet - we have only "arrange" and "act" parts, and we are missing the "assert" part of the test, - we ought to extend the current test to make it more specific. So what can we assert about the result of the `toEnglishNumber` call? We probably ought to return the string "zero", aren't we? So, let's make an appropriate assertion:

```javascript
it("converts 0 to zero", function() {
    // ARRANGE
    var number = 0;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "zero";
    expect(englishNumber).toEqual(expected);
});
```

If we run our tests right now, they will fail. We should take a careful look at the failure and see if the failure message is readable and it is what we expected. In the current situation, we will receive an assertion failure, telling us that `null` was not equal to "zero." Imagine now, that we are working on some other feature and we are not in the context of the English number conversion feature. What would be our reaction, when we run this test and see this failure? - We will probably be confused for a moment and will resort to jump to the line in the source code of the test suite that produces that error and try to understand what happened there. That is a huge context switch, and it will disrupt our current flow. So we can do better and make the failure much more readable by just adding a small thing - a cue, what that `null` was supposed to be: "english number":

```javascript
expect(englishNumber).toEqual(expected, "english number");
```

In Jasmine, the second argument to the `toEqual` matching function is the description of the failure. I think our test looks great, and so is the failure in the test output. And because we are having a good failing test, according to the second rule, we don't have to write any of test code. Now, we should make the test pass according to the third rule. And what would be the simplest way to make it work? - Return "zero":

```javascript
function toEnglishNumber(number) {
    return "zero";
}
```

At this point, we should look out for the refactoring opportunities, and I don't think there are any yet. So let's start the cycle over. To apply the first rule, we might just copy the existing test and change the input and the expected output accordingly. In that case, we would want to test another simple one-digit number - "one":

```javascript
it("converts 1 to one", function() {
    // ARRANGE
    var number = 1;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "one";
    expect(englishNumber).toEqual(expected, "english number");
});
```

Now, because the test is failing, we apply the second rule and switch back to the production code. To make the test pass, according to the third rule, we will need to introduce the simplest change possible: in that case, an "if" statement (and we will use `===` instead of `==` for comparison to avoid implicit type conversions in javascript):

```javascript
function toEnglishNumber(number) {
    if (number === 1) {
        return "one";
    }
    
    return "zero";
}
```

That is going to make all our tests pass. If we continue adding tests for the single-digit numbers while following these three rules, we will wind up with something like that:

```javascript
function toEnglishNumber(number) {
    if (number === 1) {
        return "one";
    }
    
    if (number === 2) {
        return "two";
    }
    
    if (number === 3) {
        return "three";
    }

    if (number === 4) {
        return "four";
    }
    
    if (number === 5) {
        return "five";
    }
    
    if (number === 6) {
        return "six";
    }

    if (number === 7) {
        return "seven";
    }
    
    if (number === 8) {
        return "eight";
    }
    
    if (number === 9) {
        return "nine";
    }
    
    return "zero";
}
```

At this point, we can see a clear pattern: one-to-one correspondence of a single-digit integer to the string. This code can be simplified as a pre-defined array of strings at the corresponding indices:

```javascript
var singleDigitNumbers = [
    "zero",
    "one",
    "two",
    "three",
    "four",
    "five",
    "six",
    "seven",
    "eight",
    "nine"
];
```

We also include string "zero" in that array because `return "zero"` is only happening when the number is equal to zero since we don't have any other tests right now - only from zero to nine. And the function itself will look much simpler:

```javascript
function toEnglishNumber(number) {
    return singleDigitNumbers[number];
}
```

Since we are done with the refactoring, we should proceed using the first rule of test-driven development, which means we need to write a test that fails. So let's increase the complexity of our tests and go for teen numbers. We'll start from ten:

```javascript
it("converts 10 to ten", function() {
    // ARRANGE
    var number = 10;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "ten";
    expect(englishNumber).toEqual(expected, "english number");
});
```

That will fail because so far our production code tries to fetch a string representation of the english number from the array using the number itself as an index. At the index ten, we don't have anything, so our function returns nothing. According to the second rule of test-driven development, we need to switch to production code to make it pass. There are a few ways to fix the current problem: add specific if statement to the production code, or add a string "ten" to the array. Second seems to be simpler, and we know that we can do it for the teen numbers because they can not be composed of any other small parts. So, according to the third rule, we should go for it because it is a much simpler solution:

```javascript
var singleDigitNumbers = [
    // ...
    "ten"
];
```

That makes our tests pass. And I think we have a refactoring opportunity: variable name `singleDigitNumbers` does not make any sense anymore - it contains not only single digit numbers, but, also, number "ten," which is a two-digit number. So what is common between single digit numbers and ten? They are simple numbers, i.e.: can not be composed out of other english number string representations. Let's just call them `simpleNumbers` in this case:

```javascript
var simpleNumbers = [
    // ...
];

function toEnglishNumber(number) {
    return simpleNumbers[number];
}
```

After making this refactoring, we need not forget to run the test suite to see if we didn't make a mistake. When we run it, then all our tests pass. So we can go back to the first rule of test-driven development again. Going through this cycle again for a few times will produce tests for numbers from eleven to nineteen. The production code will only have those numbers' english string representations added to the `simpleNumbers` array:

```javascript
var simpleNumbers = [
    "zero", "one", "two", "three", "four",
    "five", "six", "seven", "eight", "nine",
    
    "ten", "eleven", "twelve", "thirteen", "fourteen",
    "fifteen", "sixteen", "seventeen", "eighteen", "nineteen"
];

function toEnglishNumber(number) {
    return simpleNumbers[number];
}
```

Now it is time to introduce the concept of a complex number, such as twenty-three. It is a number that consists of the "tens" part and single-digit part. According to the first rule we have to start with the failing test, and I think we should just go for the number twenty-three:

```javascript
it("converts 23 to twenty-three", function() {
    // ARRANGE
    var number = 23;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "twenty-three";
    expect(englishNumber).toEqual(expected, "english number");
});
```

If we run our test suite, that test will fail. According to the second rule of test-driven development, now we have to switch to the production code. According to the third rule, we will have to choose the simplest code that makes this test pass (and doesn't break any other test). In our case, we have multiple options, which are similar in their simplicity. One of them is to return "twenty-three" if the number is greater than or equal to twenty:

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        return "twenty-three";
    }

    return simpleNumbers[number];
}
```

If we run our test suite, all tests will pass. Now we should see if there are any opportunities for making the code more readable and easier to understand. While the whole if statement returning a constant might feel strange, there is a concept that we can already give a name to in there: "three." We already can obtain a string "three" from number three using the method `toEnglishNumber(number)`. Let's try this refactoring:

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        return "twenty-" + toEnglishNumber(3);
    }

    return simpleNumbers[number];
}
```

That code now looks interesting. And it passes all its tests. Since, of course, we are not done yet with the implementation, according to the first rule of test-driven development, we ought to write another failing test. And we have a multitude of choices what it could be, we can just come up with other random two-digit number, such as forty-two, or we could leave the "twenty-" part in and change the "three" part to "seven," for example. Also, we could change "twenty-" part to "thirty-." Generally, in test-driven development it is better to go for the test, that will cause the smallest change to the production code, - later we will explore more on why that is. So, we could go for twenty-seven, as it will cause the smallest change to our production code:

```javascript
it("converts 27 to twenty-seven", function() {
    // ARRANGE
    var number = 27;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "twenty-seven";
    expect(englishNumber).toEqual(expected, "english number");
});
```

This test is failing, as expected. According to the second rule, we have to switch to the production code and make it pass. Also, the simplest change (third rule) that we could do is to change "3" to the last digit of the number, the remainder of the division by ten - "number % 10":

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        return "twenty-" + toEnglishNumber(number % 10);
    }

    return simpleNumbers[number];
}
```

Now if we run our test suite all the tests will pass. "The remainder of division by ten" part looks right and "twenty-" constant still feels like it is not gonna work for every two-digit number. I think it is time to write a new failing test (first rule). That will be the test that will prove that "twenty-" is not correct code. In that case, we just need to change the first digit of the number so that we could go for forty-two:

```javascript
it("converts 42 to forty-two", function() {
    // ARRANGE
    var number = 42;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "forty-two";
    expect(englishNumber).toEqual(expected, "english number");
});
```

As soon as we finish writing the assertion we will have a test failure: we are returning "twenty-two" instead of "forty-two." So it is time to switch to the production code (second rule). And we need to write just enough of it to make this test pass (third rule). We can do that by having yet another if statement:

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        if (number / 10 == 2) {
            return "twenty-" + toEnglishNumber(number % 10);
        }
        
        return "forty-" + toEnglishNumber(number % 10);
    }

    return simpleNumbers[number];
}
```

And this will make the test pass. It looks very similar to what we had with one-digit numbers, where we had an "if" statement checking that some value is equal to some number and returning an appropriate string. That is where we converted it to an array with string values, and in the function, we were fetching these strings by their index. To see if that pattern applies here we could write another similar test that will make us write another if statement:

```javascript
it("converts 39 to thirty-nine", function() {
    // ARRANGE
    var number = 39;
    
    // ACT
    var englishNumber = toEnglishNumber(number);
    
    // ASSERT
    var expected = "thirty-nine";
    expect(englishNumber).toEqual(expected, "english number");
});
```

And this fails as expected because our production code in no case can return "thirty-." So let's write the simplest if statement to make it pass. Also, to make it uniform, we would wrap "forty-" case in its appropriate if statement as a refactoring after we have a passing test suite:

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        if (number / 10 == 2) {
            return "twenty-" + toEnglishNumber(number % 10);
        }
        
        if (number / 10 == 3) {
            return "thirty-" + toEnglishNumber(number % 10);
        }
        
        if (number / 10 == 4) {
            return "forty-" + toEnglishNumber(number % 10);
        }
    }

    return simpleNumbers[number];
}
```

Certainly, there is a fair bit of duplication right now: "number / 10" and "number % 10". Let's extract them as variables. Also, let's extract "twenty", "thirty", "forty" and "toEnglishNumber(lastDigit)" parts as variables:

```javascript
function toEnglishNumber(number) {
    if (number >= 20) {
        var firstDigit = number / 10;
        var lastDigit = number % 10;
        
        var firstPart;
        if (firstDigit == 2) {
            firstPart = "twenty";
        }
        
        if (firstDigit == 3) {
            firstPart = "thirty";
        }
        
        if (firstDigit == 4) {
            firstPart = "forty";
        }
        
        var secondPart = toEnglishNumber(lastDigit);
    
        return firstPart + "-" + secondPart;
    }

    return simpleNumbers[number];
}
```

Now, we could extract the function for conversion of the first digit to the first english part, such as twenty or thirty:

```javascript
function convertTens(digit) {
    if (digit == 2) {
        return "twenty";
    }
    
    if (digit == 3) {
        return "thirty";
    }
    
    if (digit == 4) {
        return "forty";
    }
}

function toEnglishNumber(number) {
    if (number >= 20) {
        var firstDigit = number / 10;
        var lastDigit = number % 10;
        
        var firstPart = convertTens(firstDigit);
        var secondPart = toEnglishNumber(lastDigit);
    
        return firstPart + "-" + secondPart;
    }

    return simpleNumbers[number];
}
```

Now, it looks like the function `convertTens` can be simplified through usage of array in the same way as we did before with `toEnglishNumber`:

```javascript
var tens = ["", "", "twenty", "thirty", "forty"];

function convertTens(digit) {
    return tens[digit];
}
```

At this point, we can write more tests to cover all different first digits, for example fifty-seven, sixty-five, seventy-three, eighty-nine and ninety-one. To make them pass we will have to add corresponding "tens" number to our array:

```javascript
var tens = [
    "", "",
    
    "twenty", "thirty", "forty", "fifty",
    "sixty", "seventy", "eighty", "ninety"
];
```

So, that is how we apply three rules of test-driven development. Here is the full code:

```javascript
describe("toEnglishNumber", function() {

    it("converts 0 to zero", function() {
        // ARRANGE
        var number = 0;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "zero";
        expect(englishNumber).toEqual(expected);
    });

    it("converts 1 to one", function() {
        // ARRANGE
        var number = 1;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "one";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts other one-digit numbers", function() {
        expect(toEnglishNumber(2)).toEqual("two", "english number");
        expect(toEnglishNumber(3)).toEqual("three", "english number");
        expect(toEnglishNumber(4)).toEqual("four", "english number");
        expect(toEnglishNumber(5)).toEqual("five", "english number");
        expect(toEnglishNumber(6)).toEqual("six", "english number");
        expect(toEnglishNumber(7)).toEqual("seven", "english number");
        expect(toEnglishNumber(8)).toEqual("eight", "english number");
        expect(toEnglishNumber(9)).toEqual("nine", "english number");
    });

    it("converts 10 to ten", function() {
        // ARRANGE
        var number = 10;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "ten";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts other teen numbers", function() {
        expect(toEnglishNumber(11)).toEqual("eleven", "english number");
        expect(toEnglishNumber(12)).toEqual("twelve", "english number");
        expect(toEnglishNumber(13)).toEqual("thirteen", "english number");
        expect(toEnglishNumber(14)).toEqual("fourteen", "english number");
        expect(toEnglishNumber(15)).toEqual("fifteen", "english number");
        expect(toEnglishNumber(16)).toEqual("sixteen", "english number");
        expect(toEnglishNumber(17)).toEqual("seventeen", "english number");
        expect(toEnglishNumber(18)).toEqual("eighteen", "english number");
        expect(toEnglishNumber(19)).toEqual("nineteen", "english number");
    });

    it("converts 23 to twenty-three", function() {
        // ARRANGE
        var number = 23;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "twenty-three";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts 27 to twenty-seven", function() {
        // ARRANGE
        var number = 27;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "twenty-seven";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts 42 to forty-two", function() {
        // ARRANGE
        var number = 42;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "forty-two";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts 39 to thirty-nine", function() {
        // ARRANGE
        var number = 39;

        // ACT
        var englishNumber = toEnglishNumber(number);

        // ASSERT
        var expected = "thirty-nine";
        expect(englishNumber).toEqual(expected, "english number");
    });

    it("converts other two-digit numbers", function() {
        expect(toEnglishNumber(57)).toEqual("fifty-seven", "english number");
        expect(toEnglishNumber(65)).toEqual("sixty-five", "english number");
        expect(toEnglishNumber(73)).toEqual("seventy-three", "english number");
        expect(toEnglishNumber(89)).toEqual("eighty-nine", "english number");
        expect(toEnglishNumber(91)).toEqual("ninety-one", "english number");
    });

});

var simpleNumbers = [
    "zero", "one", "two", "three", "four",
    "five", "six", "seven", "eight", "nine",
    
    "ten", "eleven", "twelve", "thirteen", "fourteen",
    "fifteen", "sixteen", "seventeen", "eighteen", "nineteen"
];

var tens = [
    "", "",
    
    "twenty", "thirty", "forty", "fifty",
    "sixty", "seventy", "eighty", "ninety"
];

function convertTens(digit) {
    return tens[digit];
}

function toEnglishNumber(number) {
    if (number >= 20) {
        var firstDigit = number / 10;
        var lastDigit = number % 10;
        
        var firstPart = convertTens(firstDigit);
        var secondPart = toEnglishNumber(lastDigit);
    
        return firstPart + "-" + secondPart;
    }

    return simpleNumbers[number];
}
```

## Exercises

1. Do you see the missing edge case for two-digit numbers? Write a test for it and make it pass using three rules of test-driven development.
2. Add support for three-digit numbers.
3. Add support for negative numbers.
4. Add support for numbers with the floating point.

## Bottom Line

Today we have learned a lot of concepts from testing and test-driven development. Also, we have learned the essence of TDD - three rules of TDD. We have learned how to apply these rules on a very simple example. We have touched on how beneficial test-driven development can be when applied well.

In the next article of the series, we will discuss what different kinds of tests exist, how and when to write them and how to apply TDD in these tests. Also, we will get back to our application and implement a new feature.

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don’t hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).