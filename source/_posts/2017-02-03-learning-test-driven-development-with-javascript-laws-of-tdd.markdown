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
- Provides the safety net from bugs for the future development thanks to the fact, that no single line of production code is written other than to pass a failing test. Meaning, that whenever production code is broken in any fashion, there is a test that will point it out via failure. When adding new features or modifying the behavior of the existing ones, we might inadvertently introduce a bug. Our suite of tests will detect that bug as soon as we run all our tests, which we would certainly do during development and, especially, before the deployment or release of our systems. This is basically a near 100% test coverage*.
- Gives confidence to refactor freely and ruthlessly. Essentially, allows to change the software design of the system on the go without applying the big design upfront (BDUF). Great example: if we have a system with perfect design and without any tests, over time the design will degrade until the code becomes nearly impossible to work with; if we have a system with bad design and we have nearly 100% test coverage, over time we can improve the design of the system gradually while responding to the changes in the needs of our end users, and we will be faster as the time passes.


\* - Surely, there are subtle and complex bugs that involve whole sub-systems and 3rd-party systems to come together in some special edge case to appear - rarely, such bugs are not prevented by Test-Driven Development. Of course, we are talking about 1-3 such bugs per year - this is much better than having a bug tracker, that contains thousands of unresolved bugs. Who said it was a good idea to have a database that stores the bugs in our system? - when we apply TDD properly and our bug counts are so small, as 1-3 per year, we should treat bugs as "Red Alerts" and concentrate all the effort on shooting them down. Writing a test for these bugs when we already figured them out allows us to programmatically prove, that the system, indeed, does misbehave in a certain edge case, and prevents this bug from reocurring after we have fixed it.

---


- benefits of TDD
- example of application of 3 rules
- different types of tests
- cover concern about the speed of the test suite (how to achieve one clap of the hands or blink of an eye)


---

for the next: TDD as in Test-Driven Design
