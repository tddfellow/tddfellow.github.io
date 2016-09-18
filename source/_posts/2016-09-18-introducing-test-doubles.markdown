---
layout: post
title: "Introducing Test Doubles"
description: "Test double is a test object or a test function, that looks and behaves like its production counterpart, but is actually a simplified version that reduces the complexity and enables simpler testing."
date: 2016-09-18 16:31:48 +0200
comments: true
categories:
- javascript
- test-doubles
- testing
---

A test double is a test object or a test function, that looks and behaves like its production counterpart, but is actually a simplified version that reduces the complexity and enables simpler testing. One can represent all types of test double as an inheritance tree like this:

<img src='//g.gravizo.com/g?
@startuml;
object Double;
object Dummy;
object Stub;
object Spy;
object Mock;
object Fake;
Double <|-- Dummy;
Double <|-- Fake;
Dummy <|-- Stub;
Stub <|-- Spy;
Spy <|-- Mock;
@enduml;
'/>

Where `Double` is an abstract test double, which has no functionality - it is a general concept to talk about test doubles.

`Dummy` - is a test double, that is used to fill parameter lists, in cases where these parameters are not used by production code. Simplest `Dummy` would look like this:

```javascript
function ExampleDummyObject() {
  this.doSomething = function () {};

  this.getSomething = function () {
    return null;
  };
}

function exampleDummyFunction() {
  return null;
}

// Usage example in test
someObject.someMethod(dummyObject);
someFunction(exampleDummyFunction);
```

`Stub` - is a test dummy, additionally, providing an indirect input for the production code from the test. "Indirect" means here via a method call on the stub object or a call of the stub function. For example:

```javascript
function ExampleStubObject() {
  var something = null;

  this.getSomething = function () {
    return something;
  };

  this.stubSomething = function(somethingValue) {
    something = somethingValue;
  };
}

var someValue = null;
function exampleStubFunction() {
  return someValue;
}

// Usage example in the test
stubObject.stubSomething("a value from the test");
anObject.aMethod(stubObject);

someValue = "a value from the test";
someFunction(exampleStubFunction);
```

`Spy` - is a test stub, additionally, verifying an indirect output of the production code, by asserting afterward, without having defined the expectations before the production code is executed. For example:

```javascript
function ExampleSpyObject(assertions) {
  var didSomethingWithName = null;
  var somethingValue = null;

  this.doSomethingWith = function (name) {
    didSomethingWithName = name;
    return somethingValue;
  };

  this.stubSomething = function (something) {
    somethingValue = something;
  };

  this.assertDidSomethingWithName = function(expectedName) {
    assertions.assertTrue(
      didSomethingWithName === expectedName,
      "Expected to do something with '" + expectedName + "'"
    )
  };
}

var withName = null;
var exampleValue = null;
function exampleSpyFunction(name) {
  withName = name;
  return exampleValue;
}

function verifyExampleSpyFunction(expectedName) {
  t.assertTrue(
    withName === expectedName,
    "Expected to be called with '" + expectedName + "'"
  )
}

// Usage in the test
spyObject.stubSomething("a value from the test");
anObject.aMethod(spyObject);
spyObject.assertDidSomethingWithName("helloWorld");

exampleValue = "a value from the test";
someFunction(exampleSpyFunction);
verifyExampleSpyFunction("helloWorld");
```

`Mock` - is a stub, but the expectations are defined before the execution of the production code and it can verify itself after the execution. A simple example:

```javascript
function ExampleMockObject() {
  var expectedName = null;
  var fulfilled = false;
  var somethingValue = null;

  this.expectWillDoSomethingWithName = function (name) {
    expectedName = name;
  };

  this.doSomething = function (name) {
    assertions.assertTrue(
      name === expectedName,
      "Unexpected name '" + name + "',"
        + " expecting: '" + expectedName + "'"
    );

    fulfilled = true;
    return somethingValue;
  };

  this.stubSomething = function (value) {
    somethingValue = value;
  };

  this.verify = function () {
    assertions.assertTrue(
      fulfilled,
      "Expected to receive name '" + expectedName + "', "
        + "but got nothing"
    );
  };
}

// And the usage from the test:
mockObject.stubSomething("a value from the test");
mockObject.expectWillDoSomethingWithName("helloWorld");
anObject.aMethod(mockObject);
mockObject.verify();
```

Mocks can be much more complex (verifying order of messages, allowing multiple messages to be sent, etc.). So it is recommended to either:

- avoid them and use simpler test doubles, or
- use a full-blown well-tested mocking framework.

And if you do have to use your own custom mocks, please, write tests for them, since they can have a lot of logic inside of them.

And, finally, `Fake` - is a test double providing a simpler implementation used in the tests instead of the real thing. A good example is an in-memory database gateway, that behaves the same way the real one would, but it stores all the data in the memory. A very simple example:

```javascript
function FakeDatabase() {
  var objects = {};

  this.save = function (id, object) {
    objects[id] = object;
  };

  this.findById = function (id) {
    return objects[id];
  };

  this.findByName = function (name) {
    for (var id in objects) {
      if (objects.hasOwnProperty(id)) {
        if (objects[id].name === name) {
          return objects[id];
        }
      }
    }

    return null;
  };
}
```

Obviously, fakes require full-blown testing for them. And if the real implementation is testable (even if it is slow), it is a good idea to have the same test suite for both: fake and real implementation. This way we can really be sure, that the fake behaves the same way as the real thing. And don't forget about the edge cases, for example, if the real thing can throw a `ConnectionError`, the fake should be able too (after being instructed to do so via a special method in the tests).

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
