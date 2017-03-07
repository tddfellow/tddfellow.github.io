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

Today we are going to learn the basic principles behind the Test-Driven Development Discipline. We will take a look at the differences between Test-First and Test-Driven development. We will cover why TDD works so well, when applied skillfully. We will reveal ways to train TDD skills. And we will implement one more feature of our application.

## Basics of Test-Driven Development

A test is a single program, procedure or function, that executes our system under the test and verifies that it works as expected. System under the test is any kind of other program, part of the program or library, procedure or function. System under the test is called SUT for short.

SUT is the code that executes with a purpose of satisfying needs of our end users. Using the term "end user" we mean actual people using our system via the user interface (graphical and non-graphical), and other automated systems using our system via application programming interface (for short, API). Different term for the "end user" is "consumer of the system". Another term for the SUT is "production code" - we are going to use the latter, as it is used in Test-Driven Development much more often.

Test-Driven Development is based on a simple concept of writing production code only when there is a failing test that demands that production code for it to pass.

We call a test failing when there is some sort of an error happening when we execute it. Possible failures are the following: there is a syntax error, or code does not compile, or there is a runtime failure during test setup or production code execution, or there is an assertion error. The assertion error is the special kind of error, that happens during the final phase of the test, where we are veryfying the outcomes after running the production code under the test. An assertion error signals that the production code executed successfully, but it produced incorrect results or changed the state of the system in an incorrect fashion.

We call a test passing when there are no errors happen when we execute it. We, also, call the failing test "a red test" and we call the passing test "a green test". This is because we can not deliver the code to our customers or consumers when there are "red" tests - think of red traffic light; and we are free to go when all the tests are "green" - think of green traffic light. Most of the testing tools and frameworks format their output to present failing tests in the red color and passing tests in the green color.

Multiple tests aiming to test single SUT or a single feature of the system are usually called test suite. Depending on the context, test suite could mean that collection of tests all testing the same thing, or it could mean all tests of the entire system. For example, in the sentence "Let's read `User` class' test suite" that phrase means a collection of tests testing class `User`. On the other hand, in the sentence "Let's run the whole test suite and see if we can deploy that right now" that phrase means all tests of the entire system. The latter, sometimes, is called "suite of tests".

When the system behaves in an unexpected way and the expected behavior was previously defined or present in the code, it is called a "bug". For example, the behavior that is defined by the development team and is not implemented correctly is considered as a bug; the behavior that is defined by the development team and was implemented correctly, but now it is not working, is considered as a bug; and, finally, the behavior that was not defined may or may not be considered as a bug, depending on the produced results - if it harms or brings any value. This phenomen is called a "bug" for historical reasons: the first bug in computing was a real bug, that stuck in the computer's hardware and was causing short circuits which made the computer misbehave.

In the core of Test-Driven Development there are three steps that we need to follow:

1. We start from a failing test. We consider any kind of error a failure, including compilation and syntax errors. Meaning, that our first test for any part of the production code will fail for the reason that there is no production code to execute yet. For example: class, method or function is not defined yet.
2. Then, we resolve the failure by writing the production code. For example: in canse of missing class, we would create a class; in case of missing method, we would create a method; and in case of failing assertion error, we would fix the logic to pass the test.
3. We write no more than a simplest production code, that makes our current failing test pass, and, also, still passes all other tests.

We repeat these steps over and over until we finish the implementation of the system under the test. Strictly following these steps will lock us in a very tight loop, where we will be switching between test code and production code all the time: write one or two lines of the test code and write or change one or two lines of the production code, repeat. This cycle is, probably, twenty or thirty seconds long. It, of course, depends on how fast we can run our tests. Ideally, we want our test suite for the current system under the test to run as fast as one clap of hands, or blink of an eye.

This tight cycle gives us following benefits:

- Alerts us to the mistakes we do while coding immediately. Usually, it is enough to do one or two "Undo" commands in our editor to get back to the "green" state of the system - when all our tests are passing (occasionally, except for the last test we have added, as we will undo the incorrect implementation for it).
- Provides the safety net from bugs for the future development thanks to the fact, that no single line of production code is written other than to pass a failing test. Meaning, that whenever production code is broken in any fashion, there is a test that will point it out via failure. When adding new features or modifying the behavior of the existing ones, we might inadvertently introduce a bug. Our suite of tests will detect that bug as soon as we run all our tests, which we would certainly do during development and, especially, before the deployment or release of our systems. This is, basically, a near 100% test coverage. Surely, there are subtle and complex bugs that involve whole sub-systems and 3rd-party systems to come together in some special edge case to appear - rarely, such bugs are not prevented by Test-Driven Development. Of course, we are talking about 1-3 such bugs per year - this is much better than having a bug tracker, that contains thousands of unresolved bugs. Who said it was a good idea to have a database that stores the bugs in our system? - when we apply TDD properly and our bug counts are so small, as 1-3 per year, we should treat bugs as "Red Alerts" and concentrate all the effort on shooting them down. Writing a test for these bugs when we already figured them out allows us to programmatically prove, that the system, indeed, does misbehave in a certain edge case, and prevents this bug from reocurring after we have fixed it.
- Gives confidence to refactor freely and ruthlessly. Essentially, allows to change the software design of the system on the go without applying the big design upfront (BDUF). Great example: if we have a system with perfect design and without any tests, over time the design will degrade until the code becomes nearly impossible to work with; if we have a system with bad design and we have nearly 100% test coverage, over time we can improve the design of the system gradually while responding to the changes in the needs of our end users, and we will be faster as the time passes.
- Makes the development team always ready to deploy. While practicing test-driven development, development teams will keep latest implemented features in the source code version control system in the state, that is well-known to be "green". This state of the code can be delivered to the end users with one push of the button. This also enables teams to do continuous delivery: as soon as code is checked-in in the source code version control system (VCS), it is going to be automatically deployed to production and, essentially, be delivered to the end users.
- Promotes simplicity and alleviates the scope creep. "Oh, we don't need that yet" is the most used phrase in the vocabulary of the typical test-driven development practitioner. Tests that we write, specify only the useful functionality that we need to deliver the value to our end users. These tests don't talk about performance optimizations, security concerns, rare edge cases, "re-usable" abstractions (that, usually, are harder to understand and are difficult to re-use), etc. These things need to be implemented only when there is an actual end user value to be achieved, or the system is obviously running slower than it should, or there is an actual security concern. In that case, a very specific test (or tests) will be written to drive out that additional functionality. Since the most software development cost is in the maintainance of the existing functionality leaving out as much functionality (and code) as possible is a good idea.
- Promotes cleaner interfaces. As in test-driven development we have to write the usage example before we implement (or even design) the API in the production code, it will be optimized for ease and clarity of use and less so with the internal implementation details of that production code. Generally speaking, test-driven production code is easier to use and understand.
- Tests that call the production code serve as usage examples of that code. Essentially, in a test-driven code base, to know how a certain API can be used, it is enough to read and understand its test suite. These tests is the perfect low-level documentation for the APIs in our code base that developers can understand and read best. And that documentation is so precise that it can be executed and verified that it is still in sync with the actual code it is describing. Therefore this documentation is always up-to-date.

The most important of these benefits is the confidence to make any kind of change to the software and know in one minute or two, if that change is good to be delivered to the end user or not, with a simple push of the button. No quality assurance (QA) manual testing cycles are required. Let's take a look at the example of application of three rules of test-driven development. We will start from a very simple example, so that we don't have to touch more advanced TDD techniques.

> ### English Numbers Kata
> 
> **Given** an integer number  
> **When** I call the system with that number  
> **Then** I receive a string representation of that number in an English language
> 
> For example:
> 
> - for number `37` I receive `"thirty-seven"`
> - for number `-17451` I receive `"minus seventeen thousand four hundred fifty-one"`

According to the first rule of TDD, we have to start the implementation from the test. In test-driven development it is important to start from the simplest tests, that can be implemented via a small simple change to production code. For example: with number zero we expect result to be "zero". When writing the first test for the new functionallity we are going to design its API. In our case we are going to come up with the function name and its argument list:

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

As soon as we write `toEnglishNumber(number)` we have designed the function's signature, at least, for the single simplest case. Also, the test suite is failing now, because `toEnglishNumber` is not a function - in fact, it is undefined. This means that we have entered the red stage of test driven development and according to the second rule of TDD we have to switch back to the production code. And according to the third rule we have write just enough of it to make the failing test pass. This means writing the simplest and easiest code possible to make it pass. In this case we could just return nothing (`null` in javascript):

```javascript
function toEnglishNumber(number) {
	return null;
}
```

This is going to turn our test suite back to the green stage. At this point it is a good idea to look at both test code and production code and see if there are any opportunities for refactoring, such as: better names, extracting methods/functions, clarifying variable names, de-duplication, etc. Because we are currently in a green state we can safely apply any refactoring, automated or manual, and see if it was successful by running the test again. If, for some reason, the test is failing after the refactoring, we always have an option to CTRL/CMD+Z back to the green state, back to safety. At this point, we have finished applying one cycle of rules of TDD and we need to start over: we go back to our test code and either extend existing test to be more specific or we add more tests. Since we didn't finish writing the test yet - we have only arrange and act, and we are missing the assert part of the test, - we ought to extend the current test to make it more specific. So what can we assert about the result of the `toEnglishNumber` call? We probably ought to return the string "zero", aren't we? So, let's make an appropriate assertion:

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

If we run our tests right now, they will fail. We should take a careful look at the failure and see if the failure message is readable and it is what we expected. In the current situation, we will receive an assertion failure, telling us that `null` was not equal to "zero". Imagine now, that we are working on some other feature and we are not in the context of the english number conversion feature. What would be our reaction, when we run this test and see this failure? - We will probably be confused for a moment and will resort to jump to the line in the source code of the test suite that produces that error and try to understand what happened there. This is kind of huge context switch and it will disrupt our current context. So we can do better and make the failure much more readable by just adding a small thing - a cue, what that `null` was supposed to be: "english number":

```javascript
expect(englishNumber).toEqual(expected, "english number");
```

In Jasmine, the second argument to the `toEqual` matching function is the description of the failure. I think our test looks great and so is the failure in the test output. And because we are having a good failing test, according to the second rule, we don't have to write any of test code. Now, we should make the test pass according to the third rule. And what would be the simplest way to make it work? - Return "zero":

```javascript
function toEnglishNumber(number) {
	return "zero";
}
```

At this point, we should look out for the refactoring opportunities, and I don't think there are any yet. So let's start the cycle over. To apply the first rule, we might simply copy the existing test and change the input and the expected output accordingly. In that case, we would want to test another simple one-digit number - "one":

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

Now, because the test is failing, we apply second rule and switch back to the production code. To make the test pass, according to the third rule, we will need to introduce a simplest change possible: in that case, an if statement (and we will use `===` instead of `==` for comparison to avoid implicit type conversions in javascript):

```javascript
function toEnglishNumber(number) {
	if (number === 1) {
		return "one";
	}
	
	return "zero";
}
```

This is going to make all our tests pass. If we continue adding tests for the single-digit numbers while following these 3 rules, we will wind up with something like that:

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

At this point, we can see a clear pattern: one-to-one correspondence of a single-digit integer to the string. This code can be simplified as a pre-defined array with strings at the corresponding indices:

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

We also include string "zero" in that array because `return "zero"` is only happening when the number is equal to zero, since we don't have any other tests right now, only from zero to nine. And the function itself will look much simpler:

```javascript
function toEnglishNumber(number) {
	return singleDigitNumbers[number];
}
```

---


- example of application of 3 rules
- different types of tests
- cover concern about the speed of the test suite (how to achieve one clap of the hands or blink of an eye)


---

for the next: TDD as in Test-Driven Design
