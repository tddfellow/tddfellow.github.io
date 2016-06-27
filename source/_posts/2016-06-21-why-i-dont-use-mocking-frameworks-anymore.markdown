---
layout: post
title: "Why I don't Use Mocking Frameworks Anymore"
date: 2016-06-21 08:04:10 +0200
comments: true
categories:
- testing
- mocking
- clean-code
- architecture
- pseudo-code
---

Some time ago, I have discovered, that using your own custom test double classes, instead of a test framework makes my test code more readable and maintainable. Here is an example:

```javascript
function test_password_revealer_when_toggled_reveals_password() {
  passwordController = MockPasswordController.new()
  passwordRevealer = PasswordRevealer.new(passwordController)

  passwordRevealer.toggle()

  expect(passwordController.isRevealed()).toBeTrue()
}
```

The same test with mocking framework would look this way:

```javascript
function test_password_revealer_when_toggled_reveals_password() {
  passwordController = MockFramework.Mock.new("PasswordController")
  passwordRevealer = PasswordRevealer.new(passwordController)

  spy = spyOn(passwordController.reveal())

  passwordRevealer.toggle()

  expect(spy.haveBeenCalled()).toBeTrue()
}
```

If you take a closer look at the last example, and, specifically at these 2 lines:

```javascript
spy = spyOn(passwordController.reveal())

expect(spy.haveBeenCalled()).toBeTrue()
```

They use a language, that is not relevant to the domain, therefore, they make the test less readable.

Additionally, they have knowledge of which exactly method `PasswordRevealer#toggle()` should call on `passwordController`. If we were to rename `reveal` method, all tests for `PasswordRevealer` would fail.

Of course, creating such test doubles yourself will involve some boilerplate code - this is a trade-off. Example:

```javascript
class MockPasswordController() {
  this.state = "hidden"

  this.reveal() {
    this.state = "revealed"
  }

  this.hide() {
    this.state = "hidden"
  }

  this.isRevealed() {
    return this.state == "revealed"
  }
}
```

Making this trade-off, will simplify the case, when we were to change the name: we would change the name at 3 places:

- in test double class,
- in caller class,
- in "real" implementation class.

Whereas, with mocking framework, we would have to hunt for all failing tests, and usually, it means hundreds of failing tests.

## Thank you!

If you, my dear reader, have any thoughts, questions or arguments about the topic, feel free to reach out to me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you liked my ideas, follow me on twitter, and, even better, provide me with your honest feedback, so that I can improve.
