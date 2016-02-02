---
layout: post
title: "Test-Driven-Development Screencast #1"
date: 2016-02-02 07:59:25 +0100
comments: true
categories:
- screencast
- TDD
- test-driven-development
- tdd-screencasts
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/QnEvjrszst4?list=PLbNxoJawcer22UE8lT93-fX8ZYFNtoXFu" frameborder="0" allowfullscreen></iframe>

Recommended to watch in a full-screen and on 720p or higher quality.

## Script

Hello! I am Oleksii Fedorov, and this is the first episode of
Test-Driven-Development Screencast.

Today, I am going to briefly address following questions about
Test-Driven-Development:
- What TDD is?
- What are main benefits of doing TDD?

At the end I am going to demonstrate a small example on how to implement simple
sorting algorithm using TDD.

Let me open my slides. Don't worry: there are only three small slides.

```
vim ./slides/*
```

TDD is Software Development discipline. Therefore, basic rules, defined by TDD,
are arbitrary, weird and are to be followed to the letter, if you want it to be
useful. There is a reason for that – but we will talk shortly about that. Let
me read these basic rules for you:

1. You are not allowed to write a line of production code, unless test fails.

   Which means, that I will have to write the test even before I have something
   to test. It may sound stupid. Maybe, it is stupid. Maybe, it is not.

   But the next rule is even more weird than the first one:

2. You are not allowed to write more of a test, that is sufficient to fail.

   It is important to clarify, what ˝fail˝ means in this context. It means test
   expectation failure, and compilation/parsing/interpretation failure
   (depending if your programming language is compiled or interpreted).

   Which means, that you will have to interrupt yourself, while writing a test,
   because you have mentioned the class or package, that does not exist yet. Or
   you have mentioned the method or function, that does not exist yet.

   Now that may sound really stupid to you. Bear with me, and lets see how
   weird the last rule is:

3. You are not allowed to write more production code, that is sufficient to
   make the failing test pass.

   Which means, that once you have defined a class, that was mentioned - you
   have just fixed a failing test, and you have to go back and write more of
   the test, or add new test.

   Which means, that once you have defined a method, that was mentioned - you
   have just fixed a failing test, and you have to got back and write more of
   the test again.

   Which means, that once you changed your production code only slightly in
   direction of the correct implementation, you have to go back and write more
   tests.

   This is interesting. Now you got yourself in a very tight lock. In a very
   tight feedback loop. Write a line of test, write a line of code, write a
   line of test, write a line of code, and so on. The length of this loop is
   probably 5, 10, 30 seconds. If you have a test suite that needs half an hour
   to run, you will not do TDD.

What happens if you do not follow this discipline to the letter? If you slip
there and there?: write a bit more production code, than you had to?, write a
bit more of a test, than you had to?, or even wrote a test after writing the
whole class under test?

Well, you have just lost the main benefit of TDD: you can no longer trust the
test suite. For sure, there will be some untested code if you do it this way.

Why do we need 100% test coverage, you ask? 70% sounds like an awesome
achievement! Or is it?..

What can you tell from the fact, that only 70% of your code is covered by test
suite? Only that 30% is not covered, and therefore, there is no quick and easy
way to verify that it works.

Lets imagine the following scenario:

- You open a file on your screen.
- You spot some nasty duplication, and you know you want to fix it.
- You even see an obvious way to fix it.
- You touch your keyboard.
- And now, the fear overwhelms you: this class is not tested.
- And your reaction? - I won't touch it!

That is where code starts to rot, while nobody cleans it up, because test suite
can not be trusted, and the whole codebase slowly down-slides to a big pile of
mess.

Now, lets imagine, that you have 100% code coverage (Well, maybe 98%, because
100% is the goal, that is not achievable). And the same scenario:

- You open a file on your screen.
- You spot some nasty duplication.
- You fix the duplication.
- You run tests - and they are green.
- You check-in cleaner code in your code control system.

Or, lets say, that the problem is not trivial:
- You spot the long method.
- You split it in 3 methods - tests are still green.
- You proceed and extract these methods to the new class.
- And tests fail.
- Undo-Undo-Undo. And you are back to the green state.
- And now you think for a moment, what happened there.
- And you already have this ˝Gotcha!˝.
- And you successfully extract a class again - and the test suite is green.
- You check-in cleaner code in your code control system.

Undo button becomes your best friend. Once you stop knowing what is going on,
or what you are doing, or you simply confused, you can always go back to the
green state; that just happens to be 25 seconds ago (or 2-3 undo) away, because
of the tight feedback loop you got yourself into.

Now, there is a hidden rule of TDD. That feels more, like an implementation
detail of TDD:

```
:next
```

As tests become more specific, production code should become more generic.

And it is very true, otherwise, you would end up adding a bunch of `if`
statements every time you add a failing test.

What that means, I will point out during the example.

And lets sum up now:

100% Code Coverage =>
  Lack of Fear =>
  Consistent Random Kindness to the Code =>
  Clean Code.

---

60% Code Coverage =>
  Fear to Break It =>
  ˝I won't touch it!˝ =>
  Mess.

Now we can finally move on to the example. Next time, on the next episode of
Test-Driven-Development Screencast!

List of all TDD Screencasts can be [found here](/blog/categories/tdd-screencasts/).
