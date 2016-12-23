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

Today we are going to learn how to write tests for the whole application, that imitate real user interaction. We are going to build a small web application using Vanilla Javascript. Vanilla Javascript is a plain Javascript without any framework or library. Such tests, that imitate real user interaction via User Interface (UI) are called End-to-End Tests. These tests are the most simple to write, because we only need to think about our application in the same way user does:

- Imitating the user clicking on the button would mean for us to trigger a button click;
- Imitating the user typing text in the input field would mean for us to change input's value;
- Imitating the user clicking on the link would mean for us to trigger a link click;
- And so on.

We don't need to think about specific implementation details, such as: which functions and classes do we have in our code and how they interact with each other, is there any interaction with the back-end server or 3rd-party API.

Of course, for that kind of simplicity we are trading something off. In case of End-to-End tests, they are generally slower, suffer concurrency, wait and timeout problems, and are harder to maintain in the long run. We don't have to worry about that just yet because we want to learn how to write tests in general, and this kind of simplicity is perfect for us in this case.

Such simplicity stems from the fact that End-to-End tests, mostly, are direct translations of user stories (use case scenarios) into the UI manipulation code.

## User Story

User stories are scenarios describing certain feature of the software via user story context, sequences of user interactions, and user expectations. User story context is the description of the situation the user and the software system are in at the beginning of the scenario. Example of the system context: "user John is registered in the system with password 'welcome'". Example of the user context: "user is at the login page". User interaction is the description of the specific action user takes inside of the system, usually, doing something within the UI, for example: "User enters email 'john@example.org' in the email input field" or "User clicks on the submit button". Finally, user expectation is the description of which specific information user should receive from the system, for example: "User sees the success message on the page" or "User receives the email with the verification code".

User stories come in different flavor and formats. It can be a free-form text, describing three parts: context, interactions and expectations; or it can be in a formal "Given", "When", "Then" format.
