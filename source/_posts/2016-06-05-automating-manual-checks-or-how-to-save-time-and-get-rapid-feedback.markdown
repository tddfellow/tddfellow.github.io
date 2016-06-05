---
layout: post
title: "Automating Manual Checks or How to Save Time and Get Rapid Feedback"
date: 2016-06-05 12:37:22 +0200
comments: true
categories:
- programming
- tests
- automation
- pseudo-code
---

*It is tedious, slow and error-prone to test correctness of program manually. This article is an introduction, how to make this process as simple as a hit of one button, very fast and precise.*

In the last issue [about Iterative Divide & Conquer](/blog/2016/05/27/iterative-divide-and-conquer-or-how-to-solve-problems/) we have found out, how it is tedious to manually check that our program works correctly.

We have concluded, that it is tedious and slow to manually check that the output of our `print` statements is correct. Additionally, it is really easy to make a mistake and to oversee incorrect lines of the output, especially, when we have tons of them. As well, this applies to any other form of manual testing, including testing the application as an End User (via clicking/tapping for UI applications, via executing tons of commands for CLI applications).

## Mitigating human error-proneness via automation

Let's get back to our example from the previous issue:

```ruby
print english_integer_in_tens(20)    # twenty
print english_integer_in_tens(27)    # twenty-seven
print english_integer_in_tens(42)    # forty-two
print english_integer_in_tens(70)    # seventy
print english_integer_in_tens(99)    # ninety-nine
```

What if we could ask our computer to run `english_integer_in_tens` function with provided arguments and check that the output is exactly what we provided in a comment?

Let's try to put it in pseudo-code:

```javascript
equal(english_integer_in_tens(20), "twenty")
equal(english_integer_in_tens(42), "forty-two")
```

If we run that code, of course, we will get some sort of compilation error, or runtime error (depending on the programming language), because function `equal` is not defined yet. Let's define it! Presumably, it should be comparing left argument with right argument and printing something useful on the screen.

```javascript
function equal(left, right) {
  if (left != right) {
    print("Failure: " + left + " should be equal to " + right)
  } else {
    print("OK")
  }
}
```

If we run the program after defining this function we will get:

```
OK
OK
```

If we were to break the function `english_integer_in_tens`'s implementation, we might get something like:

```
OK
Failure: twenty should be equal to forty-two
```

## Making automation nicer

Having all this output every time we import our small library in any application going to the standard output would be annoying to say the least. How about separating the test automation from the library?

Let's extract all our testing `print`-s into the separate file, import original file from it and replace all `print`-s with `equal`-s:

```javascript
import english_integer

function equal(left, right) { ... }

equal(english_number(0), "zero")
equal(english_number(1), "one")
...
equal(english_integer(43), "forty-three")
equal(english_integer(-97), "minus ninety-seven")
```

Running only this separate file will produce expected output:

```
OK
OK
...
OK
OK
```

Importing or running directly our original library will not produce any output. This is much nicer.

## Thank you for reading!

Today we have made our testing:

- easier: hit of one button, or one command on the terminal;
- faster: from minutes of manual verification to milliseconds of automated checks;
- preciser: automated checks can't make a mistake, if we have something except `OK` - we have an error, if everything is `OK` - the program is working correctly;
- better feedback cycle: we can see if our change is correct in the matter of 1-2 seconds after making this change.

Next time we will build a fully-functional mini testing framework. Stay tuned!
