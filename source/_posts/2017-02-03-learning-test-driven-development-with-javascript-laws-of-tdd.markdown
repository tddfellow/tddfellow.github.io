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

We call a test failing when there is some sort of error happening when we execute it. Possible failures are the following: there is a syntax error, or code does not compile, or there is a runtime failure during test setup or production code execution, or there is an assertion error. The assertion error is the special kind of error, that happens during the final phase of the test, where we are veryfying the outcomes after running the production code under the test. An assertion error signals that the production code executed successfully, but it produced incorrect results or changed the state of the system in an incorrect fashion.

We call a test passing when there are no errors happen when we execute it. We, also, call the failing test "a red test" and we call the passing test "a green test". This is because we can not deliver the code to our customers or consumers when there are "red" tests - think of red traffic light; and we are free to go when all the tests are "green" - think of green traffic light. Most of the testing tools and frameworks format their output to present failing tests in the red color and passing tests in the green color.

In the core of Test-Driven Development there are three rules that we need to follow:

1. First, we write a failing test. We consider any kind of error a failure, including compilation and syntax errors. Meaning, that our first test for any part of the production code will fail for the reason that there is no production code to execute yet. For example: class, method or function is not defined yet.
2. ...


---

for the next: TDD as in Test-Driven Design
