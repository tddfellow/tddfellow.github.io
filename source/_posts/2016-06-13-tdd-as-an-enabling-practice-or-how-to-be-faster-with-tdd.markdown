---
layout: post
title: "TDD as an Enabling Practice or How to be Faster With TDD"
date: 2016-06-13 07:52:56 +0200
comments: true
<!-- published: false -->
categories:
---

Recently I had a lot of conversations with so much different programmers, with different backgrounds, contexts of work and opinions. What stroke me the most, is that majority feel that TDD is slower than simple automated testing, i.e., `TestFirst` is slower than `TestAfter`.

While digging deeper in their context of work, I could only agree with them: "True, in that context it will be about 50% slower".

Most of the time, though, the context is somewhat looking like this:

- We have some company `ACME`, that does some sort of Agile Software Development (probably Scrum);
- The company `ACME` focuses only on the business parts of Agile Software Development;
- The company `ACME` tried to introduce TDD as a development practice;
- Everything took much longer to be done.

## Enabling practice

Now, let us dive into the development practices of Agile Software Development: development practices are usually coming from Extreme Programming (XP). In XP there are 2 terms: `EnablingPractice` and `ExploitativePractice`.

Exploitative practice is the practice, that gives direct benefit to the team, such as a speed boost, or quicker feedback from users.

Enabling practice is the practice, that is required for some other exploitative practice(s) to work.

Good example of exploitative practice is Continuous Delivery. It requires Continuous Integration, Pair-Programming and Testing to be in place. These 3 are exploitative practices.
