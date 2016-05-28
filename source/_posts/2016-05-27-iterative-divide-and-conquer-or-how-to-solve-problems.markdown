---
layout: post
title: "Iterative Divide &amp; Conquer or How to solve problems"
date: 2016-05-27 07:48:18 +0200
comments: true
categories:
---

*Lets imagine a beginner programmer, who have learned a programming language basics, can write programs and can read someone else's programs. Now, for any somewhat non-trivial problem, they have trouble coming up with solution. And if they read one of the possible solutions, they will understand it and conclude: "I understand how this program works, but I don't know how to get there.". If that sounds like you, my dear reader, or someone you would like to coach or teach, then come and learn Iterative Divide & Conquer problem solving technique.*

## English Numbers Kata

Given an integer number in the range of `-999 999 999` to `999 999 999`, write a program, that will output that number in English words.

Example:

Given number `37545`, the program outputs `thirty-seven thousands five hundred forty-five`.

### I can solve that!

If you are confident, that you are able to solve that problem, my congratulations, you have problem-solving skill at a necessary level!

If you still feel, that you can solve this Kata easily, although there are more complex problems that give you troubles, then, probably, your problem-solving strategy is not scalable. The technique described in this article is scalable.

### Let's dive in

*NOTE: this article uses pseudo-code, that doesn't belong to any programming language, so it makes sense for you, my reader, to implement this step-by-step in the programming language you know.*

OK. Let's imagine, that you don't know how to solve the whole English Numbers Kata and you don't even know where to start.

Now, let's try to solve much simpler problem:

Given an integer number in the range of `0` to `9`, write a program that will output that number in English words.

Can you solve that? I am pretty sure you can. And easily at that.

Let's write down some of our possible inputs and corresponding outputs:

```ruby
print english_number(0)   # zero
print english_number(1)   # one
print english_number(7)   # seven
print english_number(9)   # nine
```

If we run that program, we will get an error, that `english_number` function is not defined yet. Let's define it:

```javascript
function english_number(number) {
  return ""
}
```

Now if we run our program, we will get four empty lines on the screen, as expected. Easiest implementation of `english_number` function would probably look like that:

```javascript
if (number == 0) {
  return "zero"
} else if (number == 1) {
  return "one"
} else if (number == 2) {
  return "two"
} else if (number == 3) {
  return "three"
} else if (number == 4) {
  return "four"
} else if (number == 5) {
  return "five"
} else if (number == 6) {
  return "six"
} else if (number == 7) {
  return "seven"
} else if (number == 8) {
  return "eight"
} else if (number == 9) {
  return "nine"
}
```

If we run our program, then we will get our expected output:

```
zero
one
seven
nine
```

If you already know arrays and access array by index, `english_number` can be simplified greatly:

```javascript
NUMBERS = ["zero", "one", "two", "three", "four",
           "five", "six", "seven", "eight", "nine"]

function english_number(number) {
  return NUMBERS[number]
}
```

After making this change, let's run the program, we should see the same output:

```
zero
one
seven
nine
```

OK. Seems like we have solved our current problem at hand. What about the initial problem? How do we get there?

### Increase size of the problem slightly

Or as you would say in real-world programming: Add a new requirement.

Our initial problem has only two axis, where we can add requirements to our current solution to move it towards final solution:

- increase supported range
- allow negative integers

Let's go with latter, it seems easier. First we write down our possible inputs and outputs:

```ruby
print english_integer(0)     # zero
print english_integer(4)     # four
print english_integer(-3)    # minus three
print english_integer(9)     # nine
print english_integer(-9)    # minus nine
```

Now, implement `english_integer` as a simple call to `english_number` (that will make half of our inputs produce correct output):

```javascript
function english_integer(number) {
  return english_number(number)
}
```

Depending on what your programming language this can:

- output partly incorrect data (for negative values)
- fail at run time
- fail at compile time

We can fix that by handling the case of negative numbers:

```javascript
if (number < 0) {
  return "minus " + english_integer(-number)
} else {
  return english_number(number)
}
```

That might be confusing at first. Especially, part, where we call `english_integer` from the body of the same function. Let's draw a diagram on how that function works:

![Diagram for english_integer after adding negative integers](https://camo.githubusercontent.com/134fbbce495dc2d9bb3f137077cdde57dc28668e/687474703a2f2f672e67726176697a6f2e636f6d2f673f2530412532302532306469677261706825323047253230253742253041253230253230253230253230666f7263656c6162656c733d747275653b253041253230253230253230253230253232656e676c6973685f696e7465676572286e756d6265722925323225323025354273686170653d626f782535443b2530412532302532302532302532302532326966253230286e756d626572253230253343253230302925323225323025354273686170653d6469616d6f6e642535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532323b25304125323025323025323025323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532323b25304125323025323025323025323025323270726570656e64253230726573756c7425323077697468253230276d696e7573253230272532323b253041253230253230253230253230253232616e6425323072657475726e25323069742532323b2530412532302532302532302532302532327468656e25323072657475726e253230726573756c742532306f662532307468697325323063616c6c2532323b253041253230253230253230253230253232656e676c6973685f696e7465676572286e756d626572292532322532302d2533452532302532326966253230286e756d62657225323025334325323030292532323b2530412532302532302532302532302532326966253230286e756d62657225323025334325323030292532323a652532302d25334525323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532322532302535426c6162656c3d2532325945532532322535443b2530412532302532302532302532302532326966253230286e756d62657225323025334325323030292532323a772532302d25334525323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532322532302535426c6162656c3d2532324e4f2535436e696e76617269616e743a2535436e6e756d6265722532302533453d253230302532322535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532323a6e652532302d253345253230253232656e676c6973685f696e7465676572286e756d626572292532323a6e652532302535426c6162656c3d253232696e76617269616e743a2535436e6e756d6265722532302533453d253230302532322535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532322532302d25334525323025323270726570656e64253230726573756c7425323077697468253230276d696e7573253230272532322532302d253345253230253232616e6425323072657475726e25323069742532323b25304125323025323025323025323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532322532302d2533452532302532327468656e25323072657475726e253230726573756c742532306f662532307468697325323063616c6c2532323b253041253230253230253744)

If we were to unwind this diagram into possible paths, we would end up with two possible cases:

1. When `number < 0`:
   - `english_integer(number)`
   - `if (number < 0)` - YES
   - `english_integer(-number)`
   - `if (number < 0)` - NO
   - `english_number(number)`
   - prepend result of last call with `minus `
   - and return it
2. When `number >= 0`:
   - `english_integer(number)`
   - `if (number < 0)` - NO
   - `english_number(number)`
   - return result of last call

Running this program results in expected output:

```
zero
four
minus three
nine
minus nine
```

Implementation of `english_integer(number)` function can be simplified by eliminating `else` clause and treating `number < 0` as an edge case and using "guard clause" to handle it:

```javascript
function english_integer(number) {
  if (number < 0) return "minus " + english_integer(-number)

  return english_number(number)
}
```

If we run our program, output should be the same as before.

### Add more requirements!

Now, that we are done with handling negative case, we have only one requirement axis left: range of integers. Currently we support integers from `-9` to `9`. Now let's extend support for integers from `10` to `19`. This should be as trivial as solving problem for range of `0...9`.

Our inputs with corresponding outputs:

```ruby
print english_teen_number(10)    # ten
print english_teen_number(11)    # eleven
print english_teen_number(13)    # thirteen
print english_teen_number(18)    # eighteen
print english_teen_number(19)    # nineteen
```

And the implementation to make the output of the program correct (very similar to `english_number` function):

```javascript
TEEN_NUMBERS = ["ten", "eleven", "twelve", "thirteen", "fourteen",
                "fifteen", "sixteen", "seventeen", "eighteen", "nineteen"]

function english_teen_number(number) {
  // "number - 10" comes from the fact, that range 10..19
  // maps to a range of 0..9 of our array, so we need to
  // shift 10..19 by -10 to get 0..9
  return TEEN_NUMBERS[number - 10]
}
```

Now we want to glue our current solution with `english_teen_number` to add support for range `10..19`. This sounds like another `if .. else` case handling:

- when `number < 10`, use `english_number`
- when `number >= 10`, use `english_teen_number`

Let's change our input/output pairs for `english_integer` function:

```ruby
print english_integer(0)     # zero
print english_integer(4)     # four
print english_integer(-3)    # minus three
print english_integer(9)     # nine
print english_integer(-9)    # minus nine
print english_integer(10)    # ten
print english_integer(15)    # fifteen
print english_integer(19)    # nineteen
```

That should fail in some way (compile error, run time error or incorrect output). Now it is time to implement missing functionality for last 3 examples in our `english_integer` function. Replace last call to `english_number` with:

```javascript
if (number < 10) {
  return english_number(number)
} else {
  return english_teen_number(number)
}
```

If we run the program, we should see correct output:

```
zero
four
minus three
nine
minus nine
ten
fifteen
nineteen
```

What about negative numbers in range of `-19...-10`? Let's add input/output examples and see what happens if we run the program:

```ruby
print english_integer(-12)    # minus twelve
print english_integer(-16)    # minus sixteen
print english_integer(-19)    # minus nineteen
```

And the output if we run the program:

```
minus twelve
minus sixteen
minus nineteen
```

Yes, it works already thanks to the guard statement that we have at the top of `english_integer` function. And the whole function body:

```javascript
function english_integer(number) {
  if (number < 0) return "minus " + english_integer(-number)

  if (number < 10) {
    return english_number(number)
  } else {
    return english_teen_number(number)
  }
}
```

### Going further!

Next smallest requirement seems to be an addition of the range `20...99`. Let's write down our example inputs and outputs:

```ruby
print english_integer_in_tens(20)    # twenty
print english_integer_in_tens(27)    # twenty-seven
print english_integer_in_tens(42)    # forty-two
print english_integer_in_tens(70)    # seventy
print english_integer_in_tens(99)    # ninety-nine
```

So how do we solve this problem? Can we follow the same pattern as before, for ranges `0...9` and `10...19`?

We certainly do. We create a constant `INTEGERS_IN_TENS = [...]`, where `[...]` will contain 79 (mostly two-word) strings. It seems like an excess effort to me. So can we do better?

Yes! We can apply the same problem-solving technique here. What is the smallest problem, that we can solve here easily and independently from other problems?

What about solving the problem only for numbers `20`, `30`, `40`, ..., `90`? Sounds simple enough! Let's write our example input/outputs:

```ruby
print english_ten(20)    # twenty
print english_ten(30)    # thirty
print english_ten(60)    # sixty
print english_ten(90)    # ninety
```

This is no longer different from ranges `0...9` and `10...19`. The only detail, is that we need to find a way to convert range `20...90` to `0...7` to be able to access the array by that index. This can be done in 2 steps:

1. obtain first digit of the number: `number / 10`, which results in the range `2...9`,
2. and shift resulting digit by `-2`: `number / 10 - 2`, which results in the range `0...7`,

exactly, what we expect. Now, we can implement `english_ten` similarly to `english_number` and `english_teen_number`:

```javascript
TENS = ["twenty", "thirty", "forty", "fifty",
        "sixty", "seventy", "eighty", "ninety"]

function english_ten(number) {
  return TENS[number / 10 - 2]
}
```

Now it is time to return to the original requirement: support range `20...99`. Looking at our example input/outputs for a function `english_integer_in_tens`, we can conclude, that we have 2 different cases:

- When second digit is 0 (`number % 10 == 0`):
  - we output only `twenty`, `thirty`, ..., `ninety`, depending on first digit of the `number`;
  - by the way, this is exactly, what `english_ten` function is doing. Great!
- When second digit is not 0:
  - we output `twenty`, `thirty`, ..., `ninety`, depending on first digit of the `number`;
  - and we output a hyphen character: `-`;
  - and we output `one`, `two`, ..., `nine`, depending on second digit;
  - by the way latter is exactly, what `english_number` function is doing. Great!

So, let's implement that in `english_integer_in_tens`:

```javascript
function english_integer_in_tens(number) {
  second_digit = number % 10

  if (second_digit == 0) {
    return english_ten(number)
  } else {
    return english_ten(number) + " " + english_number(second_digit)
  }
}
```

Running the program will produce the expected output:

```
twenty
twenty-seven
forty-two
seventy
ninety-nine
```

Now we should glue the solution for range `20...99` with our main solution, that currently supports only `-19...19`. As always, start with example input/outputs:

```ruby
print english_integer(60)   # sixty
print english_integer(43)   # forty-three
print english_integer(-97)  # minus ninety-seven
```

This results in incorrect output or failure. How can we merge two solutions together now? Let's remember our current main solution's different cases:

- Guard `number < 0`, that prepends "minus " and makes number non-negative.
- When `number < 10`, use `english_number`.
- Otherwise, use `english_teen_number`.

Seems, like first two cases should not be touched and we are interested in the last one. We should split it in two:

- Otherwise:
  - When `number < 20`, use `english_teen_number`.
  - Otherwise, use `english_integer_in_tens`.

I believe, we are done and can implement merged version:

```javascript
function english_integer(number) {
  if (number < 0) return "minus " + english_integer(-number)

  if (number < 10) {
    return english_number(number)
  } else if (number < 20) {
    return english_teen_number(number)
  } else {
    return english_integer_in_tens(number)
  }
}
```

Running the program confirms that our new solution now supports integers in the range `-99...99`.

### Final solution

I'm leaving the final solution as an exercise to you, my dear reader. There is nothing more to this technique. Careful application of this technique and careful choice of smallest baby-steps as new requirements to your current solution will get you there - to the final solution, that supports the range of `-999 999 999...999 999 999`.

If you have any troubles, don't hesitate to share your questions, ideas and code with me. I will try my best to help you, my dear reader. I am reachable via Twitter by handle [@waterlink000](https://twitter.com/waterlink000).

## Challenge #1

Extend solution to support floating-point numbers to a precision level of 6 digits after the dot.

## Challenge #2

Come up with completely generic solution, that is capable of working with any number, that can be represented on a computer. Including `Big Integer` or `Big Decimal` if your programming language supports these concepts.

## Thank you for reading!

Final note: did you notice, how tedious it is to check, that output of all these `print`-s is correct, each time you run the program? This can and should be automated! Stay tuned, I am going to publish article on how to easily automate these checks really soon!
