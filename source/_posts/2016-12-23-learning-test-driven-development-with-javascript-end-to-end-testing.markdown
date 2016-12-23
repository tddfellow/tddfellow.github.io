---
layout: post
title: "Learning Test-Driven Development with Javascript: End-to-End Testing"
date: 2016-12-23 10:13:33 +0100
comments: true
categories:
- tdd
- tutorial
- learning-tdd-with-js
- javascript
---

**Level: Beginner.**

Today we are going to learn how to write tests for the whole application, that imitate real user interaction. We are going to build a small web application using Vanilla Javascript. Vanilla Javascript is a plain Javascript without any framework or library. Such tests, that imitate real user interaction via user interface (UI) are called End-to-End Tests. These tests are the most simple to write, because we only need to think about our application in the same way user does:

- Imitating the user clicking on the button would mean for us to trigger a button click;
- Imitating the user typing text in the input field would mean for us to change input's value;
- Imitating the user clicking on the link would mean for us to trigger a link click;
- And so on.

We don't need to think about specific implementation details, such as: which functions and classes do we have in our code and how they interact with each other, is there any interaction with the back-end server or 3rd-party API.

Of course, for that kind of simplicity we are trading something off. In case of End-to-End tests, they are generally slower, suffer concurrency/wait/timeout problems and are harder to maintain in the long run. We don't have to worry about that just yet, since we want to learn how to write tests in general, and this kind of simplicity is perfect for us.
