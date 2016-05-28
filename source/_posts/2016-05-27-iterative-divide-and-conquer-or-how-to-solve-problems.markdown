---
layout: post
title: "Iterative Divide &amp; Conquer or How to solve problems"
date: 2016-05-27 07:48:18 +0200
comments: true
categories:
---

*Lets imagine a beginner programmer, who have learned a programming language basics, can write programs and can read someone else's programs. Now, for any somewhat non-trivial problem, they have trouble coming up with solution. And if they read one of the possible solutions, they will understand it and conclude: "I understand how this program works, but I don't know how to get there.". If that sounds like you, my reader, or someone you would like to coach or teach, then come and learn Iterative Divide & Conquer problem solving technique.*

## English Numbers Kata

Given an integer number in the range of `-999 999 999` to `999 999 999`, write a program, that will output that number in English words.

Example:

Given number `37545`, the program outputs `thirty seven thousands five hundred forty five`.

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

![Diagram for english_integer after adding negative integers](https://camo.githubusercontent.com/dbbbe1088cb0ad08b0b9bbc6fb64bbd8de66f614/687474703a2f2f672e67726176697a6f2e636f6d2f673f2530412532302532306469677261706825323047253230253742253041253230253230253230253230253232656e676c6973685f696e7465676572286e756d6265722925323225323025354273686170653d626f782535443b2530412532302532302532302532302532326966253230286e756d626572253230253343253230302925323225323025354273686170653d6469616d6f6e642535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532323b25304125323025323025323025323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532323b25304125323025323025323025323025323270726570656e64253230726573756c7425323077697468253230276d696e7573253230272532323b253041253230253230253230253230253232616e6425323072657475726e25323069742532323b2530412532302532302532302532302532327468656e25323072657475726e253230726573756c742532306f662532307468697325323063616c6c2532323b253041253230253230253230253230253232656e676c6973685f696e7465676572286e756d626572292532322532302d2533452532302532326966253230286e756d62657225323025334325323030292532323b2530412532302532302532302532302532326966253230286e756d62657225323025334325323030292532322532302d25334525323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532322532302535426c6162656c3d2532325945532532322535443b2530412532302532302532302532302532326966253230286e756d62657225323025334325323030292532322532302d25334525323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532322532302535426c6162656c3d2532324e4f2535436e696e76617269616e743a2535436e6e756d6265722532302533453d253230302532322535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532322532302d253345253230253232656e676c6973685f696e7465676572286e756d626572292532322532302535426c6162656c3d253232696e76617269616e743a2535436e6e756d6265722532302533453d253230302532322535443b25304125323025323025323025323025323263616c6c253230656e676c6973685f696e7465676572282d6e756d626572292532322532302d25334525323025323270726570656e64253230726573756c7425323077697468253230276d696e7573253230272532322532302d253345253230253232616e6425323072657475726e25323069742532323b25304125323025323025323025323025323263616c6c253230656e676c6973685f6e756d626572286e756d626572292532322532302d2533452532302532327468656e25323072657475726e253230726573756c742532306f662532307468697325323063616c6c2532323b253041253230253230253744)

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
