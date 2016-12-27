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

"Learning TDD with Javascript" is the series of articles where we learn basics of automated testing and test-driven development. While language of choice for the code examples is Javascript, all described concepts are language-agnostic and can be applied in various technological stacks. In these articles reader is expected to do small exercises after each major topic to make sure that theoretical knowledge is reinforced with practice. Some of these exercises are practical and will involve coding, or simple writing; others will be a food for a thought. Also, reader might want to get their feedback on these exercises, so don't hesitate to send the results my way: oleksii@tddfellow.com - feedback on the practice is quite important as it helps to improve quicker, when you know what is well and what can be improved and how. Also, don't hesitate to send any questions and feedback regarding the content of these articles. Your questions, feedback and your practical results will help authors shape this content better.

Today we are going to learn how to write tests for the whole application, that imitate real user interaction. We are going to build a small web application using Vanilla Javascript. Vanilla Javascript is a plain Javascript without any framework or library. Such tests, that imitate real user interaction via User Interface (UI) are called End-to-End Tests. These tests are the most simple to write, because we only need to think about our application in the same way user does:

- Imitating the user clicking on the button would mean for us to trigger a button click;
- Imitating the user typing text in the input field would mean for us to change input's value;
- Imitating the user clicking on the link would mean for us to trigger a link click;
- And so on.

We don't need to think about specific implementation details, such as: which functions and classes do we have in our code and how they interact with each other, is there any interaction with the back-end server or 3rd-party API.

Of course, for that kind of simplicity we are trading something off. In case of End-to-End tests, they are generally slower, suffer concurrency, wait and timeout problems, and are harder to maintain in the long run. We don't have to worry about that just yet because we want to learn how to write tests in general, and this kind of simplicity is perfect for us in this case.

Such simplicity stems from the fact that End-to-End tests, mostly, are direct translations of user stories (use case scenarios) into the UI manipulation code.

## User Story

User stories are scenarios describing certain feature of the software via user story context, sequences of user interactions and user expectations. User story context is the description of the situation the user and the software system are in at the beginning of the scenario. Example of the system context: "user John is registered in the system with password 'welcome'". Example of the user context: "user is at the login page". User interaction is the description of the specific action user takes inside of the system, usually, doing something within the UI, for example: "User enters email 'john@example.org' in the email input field" or "User clicks on the submit button". Finally, user expectation is the description of which specific information user should receive from the system, for example: "User sees the success message on the page" or "User receives the email with the verification code".

User stories come in different flavor and formats. It can be a free-form text, describing three parts: context, interactions and expectations; or it can be in a formal "Given", "When", "Then" format. "Given" part is the sequence of the user story context descriptions, "When" part is the sequence of the user interaction descriptions, and "Then" part is the sequence of user expectation descriptions. Both forms can be used interchangeably and some software companies use one or another formal version of the user story format consistently.

Let's take a look at the example of the free-form user story together with the example of the Given-When-Then user story for the same feature:

<div class="next-table-layout-is-fixed"></div>
|Free-form|Given-When-Then|
|-|-|
|<hr>|<hr>|
|User with email 'john@example.org' and password 'welcome' exists in the system<br>John enters his email 'john@example.org' into the email field and his password 'welcome' into the password field on the login page<br>After that John submits the login form<br>Finally, John expects to see their profile page with the indication of him being logged in (name 'John' is present on the page)|**Given** User with email 'john@example.org' and password 'welcome' exists<br>**And** I am at the login page<br>**When** I enter 'john@example.org' in the email field<br>**And** I enter 'welcome' in the password field<br>**And** I click on the submit button<br>**Then** I see the profile page<br>**And** I see my name as the title of the profile page|

As we can see, free-form can be very vague and is very flexible and formal form is more strict and specific. The free-form on its own doesn't have much upsides or downsides - it is as good as it was written. On the other hand, formal form does give us some value and also trades something off for that value: they are generally easier to write and, because they are so specific, are easier to translate to the automated test, although they may hamper creativity either while creating the user story or when implementing it.

It is important to mention, that these use case scenarios are not full user stories or features. One feature can have multiple scenarios like that - together they are called acceptance criteria. When all such scenarios of the given feature work correctly, the feature is done. There is another vital part of the user story - general description, that should contain the rationale behind the story and the value for the user or any other important actor in the system, such as the stakeholder. Before we write scenarios, we often come up with the rationale like that and it drives us to write a scenario. For example, for the feature above we would have used something like that:

<div class="next-table-layout-is-fixed"></div>
|Free-form|Given-When-Then|
|-|-|
|<hr>|<hr>|
|John needs to authenticate to the system so that he can access his private content|**As John**, I want to be able to authenticate to the system<br>**So that** I can access my private content|

Because formal form user stories are more precise, easier to write and easier to translate to the automated test, we are going to use them to learn End-to-End testing in the context of Test-Driven Development.

### Exercises

1. Write a scenario for authentication story, when John enters wrong password.
2. Write a user story with rationale and scenarios for the sign-up feature.
3. Imagine we are developing an instant messaging application. What do you think would be the next feature after login/sign-up? Write the user story with rationale and scenarios for this feature.

Do you have questions? Or do you want to get a quick feedback on how you did the exercises? - mail me: oleksii@tddfellow.com

## Setting Up the Project

Now we will be writing a simple web application, using Vanilla Javascript (ECMAScript 5), so that our setup is rather simple. For the testing we will be using standalone version of the Jasmine testing framework. Also, we will be writing a single page application (SPA), so that we don't have to worry about rendering different pages in our tests for now. We are aiming for the following directory structure of our project:

```
.
├── lib
├───── .. external libraries, such as Jasmine ..
├── spec
├───── .. sources with Jasmine automated tests ..
└── src
├───── .. sources with our main code ..
├── index.html       - entry point to the application
├── SpecRunner.html  - entry point to our test suite
```

First, create required directories: `lib`, `spec` and `src`. Then download the latest standalone Jasmine release here: https://github.com/jasmine/jasmine/releases (jasmine-standalone-{version}.zip file). At the time of writing, the version is `2.5.2`. Unzip that file into your project directory. You should get the following files from it:

- `./lib/jasmine-2.5.2/` directory - contains all the resources required by Jasmine.
- `./SpecRunner.html` - example entry point to our test suite.
- `./src/Player.js` and `./src/Song.js` - example source files.
- `./spec/PlayerSpec.js` and `./spec/SpecHelper.js` - example automated Jasmine tests.

Now, try to open `SpecRunner.html` in your browser. It should run these example tests and they should all pass. Here is how it should look like:

![Jasmine example test run with all tests green](/images/learning-tdd-with-js/jasmine-example-run.png)

Also, create an empty `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>
```

And now we should have the desired project structure. So how does that Jasmine testing framework works, anyways?

## Crash Course into Jasmine

Let's take a look at the example test file to get the gist of how Jasmine works:

```javascript
// spec/PlayerSpec.js

describe("Player", function() {
  var player;
  var song;

  beforeEach(function() {
    player = new Player();
    song = new Song();
  });

  it("should be able to play a Song", function() {
    player.play(song);
    expect(player.currentlyPlayingSong).toEqual(song);

    //demonstrates use of custom matcher
    expect(player).toBePlaying(song);
  });

  describe("when song has been paused", function() {
    beforeEach(function() {
      player.play(song);
      player.pause();
    });

    it("should indicate that the song is currently paused", function() {
      expect(player.isPlaying).toBeFalsy();

      // demonstrates use of 'not' with a custom matcher
      expect(player).not.toBePlaying(song);
    });

    it("should be possible to resume", function() {
      player.resume();
      expect(player.isPlaying).toBeTruthy();
      expect(player.currentlyPlayingSong).toEqual(song);
    });
  });

  // demonstrates use of spies to intercept and test method calls
  it("tells the current song if the user has made it a favorite", function() {
    spyOn(song, 'persistFavoriteStatus');

    player.play(song);
    player.makeFavorite();

    expect(song.persistFavoriteStatus).toHaveBeenCalledWith(true);
  });

  //demonstrates use of expected exceptions
  describe("#resume", function() {
    it("should throw an exception if song is already playing", function() {
      player.play(song);

      expect(function() {
        player.resume();
      }).toThrowError("song is already playing");
    });
  });
});
```

First important concept here is the `describe("...", function () { ... })`. `describe` function is used to describe a certain concept or a certain context. For example, `describe("Player", ...)` means that we are going to define tests for `Player` class or some other `Player` concept. Also, describes can be nested to indicate that we are describing some special context (`describe("when song has been paused", ...)`) or sub-concept of current concept, such as the method of currently described class (`describe("#resume", ...)`). This is a good example, of what the unit test suite might be describing. In the case of End-to-End tests we would like to describe whole feature, so `describe("Login Feature", ...)` is a good bet. The second argument for the `describe` is the function, that will contain all tests and sub-describes for the described concept. This function is the capturing closure, so defining variables and/or functions on the outer-level describe will make them available on the inner-level describes and the tests themselves.

Second important concept here is the `it("...", function () { ... })`. `it` function is used to create a test for the currently described concept, context or sub-concept. First argument is the description of what *it* does, where *it* is the described concept. For example, given we describe a `music player` and our context is `when volume is at max`, then we might write the test `it("is very loud", ...)`. In the case of End-to-End tests we are describing a feature, so *it* will refer to our application, or application's user interface, for example: `it("shows user's nickname", ...)`. The second argument to the `it` is the function with the test itself. Here we will setup the stage for the test, call our main code and verify that everything happened as we expect.

Finally, third important concept is the `expect(...).to...`. This is the Jasmine's form of assertion. This is where we verify that our code worked as we expect it to. As an argument to expect we provide an actual value: something that our code has returned as a result of the function or method execution, or something that we have read from the UI using UI manipulation code, or something that we have read from some sort of 3rd party service, such as our own back-end server, 3rd-party API or database. Essentially, this is the value that we are verifying to be correct. The second part is the Jasemine matcher - the method defined on the object, that is returned from `expect(value)` call, that allows us to define what we want to assert about that value. The most used one is `toEqual(...)`, which simply asserts that the value was equal to some expected value.

These 3 concepts should be enough to start writing tests. Don't worry about everything else that you see in this example file from standalone Jasmine distribution. We will discover some of these concepts as we go. Now, let's remove `src/Player.js`, `src/Song.js`, `spec/PlayerSpec.js` and `spec/SpecHelper.js`, and run our test suite - it is enough to reload the page to re-run our test suite. Test run should report that `No specs found`:

![Jasmine no specs found example](/images/learning-tdd-with-js/jasmine-no-specs-found.png)

To get the gist of how `describe("...", function () { ... })`, `it("...", function () { ... })` and `expect(...).toEqual(...)` works, let's write our first failing test. Also, that will let us see if the testing framework is configured correctly and is capable of showing us the test failure. Let's create a new file called `spec/JasmineWorksSpec.js`:

```javascript
// spec/JasmineWorksSpec.js

describe("testing framework", function () {
  it("works", function () {
    expect(2 + 2).toEqual(5);
  });
});
```

And we need to add this test file to the `SpecRunner.html`:

```html
<!-- include source files here... -->

<!-- include spec files here... -->
<script src="spec/JasmineWorksSpec.js"></script>
```

And if we run our test suite, we should see a failure:

```
testing framework works
  Expected 4 to equal 5.
```

And we ought to make it pass by fixing our incorrect assertion: `expect(2 + 2).toEqual(4);`. And if we run the test suite again, by reloading the page in the browser, all the test should pass.

## Writing our First Simple Tests

Now we can write some real tests to practice usage of `describe`, `it` and `expect(...).toEqual`: let's create `ArithmeticsSpec.js` and write some tests for behavior of `add` function:

```javascript
// spec/ArithmeticsSpec.js

describe("Arithmetics", function () {

  describe("#add(a, b)", function () {
    it("calculates the sum of two numbers", function () {
      // ARRANGE
      var a = 3;
      var b = 4;
      var expected = 7;

      // ACT
      var actual = Arithmetics.add(a, b);

      // ASSERT
      expect(actual).toEqual(expected);
    });
  });

});
```

Don't forget to add the `<script src="spec/ArithmeticsSpec.js"></script>` to the `SpecRunner.html`. This test will fail first because `Arithmetics` module is not defined. We will define it as an empty object. Next failure is because `Arithmetics.add` is not a function - it is not defined. We will define that function with two arguments inside of the `Arithmetics` object. Finally, test will fail, because we expect the result to be seven, but it was undefined. We will make the simplest thing we can do to pass the failing test - return seven. This will make the test pass. The code will be in `src/Arithmetics.js`, which we include in our `SpecRunner.html`, and will look like that:

```javascript
// src/Arithmetics.js

var Arithmetics = {
  add: function (a, b) {
    return 7;
  }
};
```

That, of course, is not correct implementation, so we need another test to drive out the correct implementation - test with different inputs and different result. To make it pass we will have to use `a + b`:

```javascript
// spec/ArithmeticsSpec.js
describe("Arithmetics", function () {

  describe("#add(a, b)", function () {
    it("calculates the sum of two numbers", function () { ... });

    it("calculates the sum of two other numbers", function () {
      // ARRANGE
      var a = 73;
      var b = 89;
      var expected = 162;

      // ACT
      var actual = Arithmetics.add(a, b);

      // ASSERT
      expect(actual).toEqual(expected);
    });
  });

});

// src/Arithmetics.js
var Arithmetics = {
  add: function (a, b) {
    return a + b;
  }
};
```

### Three "A"s: Arrange, Act and Assert

Have you noticed three comments that I have left in the example tests' code: `ARRANGE`, `ACT` and `ASSERT`? These are three "A"s of writing a good test. Arrange is the part of the test, where we set up the stage: prepare input data, create objects, load resources, change state of the system - it is the part where we create the context for our test. Act is the part of the test, where we call our system under the test. In the `Arithmetics` example it was function `Arithmetics.add(a, b)`. System under the test can return some useful value or change the its own state. To verify that either is correct we, finally, use Assert section of our test - part of the test where we verify the outcome of the call to the system under the test.

The sections `ARRANGE`, `ACT` and `ASSERT` are spelled out in the comments only for the reader's convenience - usually, real projects don't have such comments. It is worth noting that for learning and practicing purposes it is a good idea to name these sections explicitly in our tests - this develops a habit of recognizing which part of the test should belong to which section. Also, the formula Act -> Arrange -> Assert makes it easier to come up with the test when we lack deep experience in testing, or with particular testing framework, or environment.

Going a bit back to our user story scenarios: have you noticed the connection between "Arrange, Act and Assert" and "Given-When-Then"? Arrange part of the test corresponds to the Given part of the scenario, Act part of the test corresponds to the When part of the scenario, and Assert part of the test corresponds to the Then part of the scenario. Let's see it on of our previous example scenarios:

<div class="next-table-is-top-aligned"></div>
|Test Section|Scenario Step|
|-|-|
|<hr>|<hr>|
|ARRANGE|**Given** User with email ‘john@example.org’ and password ‘welcome’ exists<br>**And** I am at the login page|
|<br>|<br>|
|ACT|**When** I enter ‘john@example.org’ in the email field<br>**And** I enter ‘welcome’ in the password field<br>**And** I click on the submit button|
|<br>|<br>|
|ASSERT|**Then** I see the profile page<br>**And** I see my name as the title of the profile page|

### Exercises

1. Write tests and implement functions on `Arithmetics` module: `subtract`, `multiply` and `divide`.
2. Inline Arrange, Act and Assert in one-liner. Can you still recognize implicit Arrange, Act and Assert sections in that one line? Is it more or less readable? Is there a middle ground between two versions? Why would you choose one or the other?
3. Classify parts of the free-form story for the user log in from before as Arrange, Act and Assert. Was it easier, than classifying formal Given-When-Then form? Was it harder? Or maybe the same? How would it be if you never saw Given-When-Then version of that free-form story?
