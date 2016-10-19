---
layout: post
title: "Do More With Baby-Steps TDD"
description: "I'm usually advocating for the Baby-Steps Test-Driven Development with Triangulation. On the first encounter, this technique seems very verbose and everybody wonders how can it possibly work and why I am very productive with it. Let me tell you about that. First, let's quickly recap both techniques..."
date: 2016-10-19 23:15:12 +0200
comments: true
categories:
- incremental
- triangulation
- tdd
- baby-steps
- willpower
---

Hello everyone! I'm usually advocating for the Baby-Steps Test-Driven Development with Triangulation. On the first encounter, this technique seems very verbose and everybody wonders how can it possibly work and why I am very productive with it. Let me tell you about that. First, let's quickly recap both techniques:

## Baby-Steps TDD

In Baby-Steps TDD the basic strategy is to get to the green state ASAP. If you can pass all tests with `return 42`, you should! While the benefit of the approach is not directly obvious exploring the alternative shows its value. One possible alternative is to write a bunch of tests for the software and then make them all pass. This results in a lot of changes made to the software under the test while tests are failing (are in red state). This provides very slow feedback and high risks because with every decision in the code complexity grows exponentially and problems are hard to find when you only know that software worked 1 hour ago and there is one mistake in a whole 1 hour worth of work. The same effect can be seen if the most complex test is written first so that it forces the engineer to implement the whole solution or big part of it in one go.

Baby-Steps TDD mitigates the issue by ensuring everything worked one or two minutes ago. At least to the extent the software is specified by currently written tests. So if something does not work as expected it is most probably a mistake in these last 2-3 lines of code that we have written. And we can even discard them entirely with "undo" command and start over from the green state without losing much work and saving a whole lot of time debugging.

Baby-Steps TDD provides faster feedback and fewer risks for the cost of a bit more overall effort. Let's take a look into the triangulation technique:

## Triangulation Technique

In essence, Triangulation Technique takes ideas of Baby-Steps TDD further and reduces step size even further. For example, when usually with Baby-Steps TDD you would need one test to introduce the correct `if` statement, with Triangulation Technique and Baby-Steps TDD combined you would use multiple tests for this:

- The first test for the most degenerated case which requires one to write a simple `return CONSTANT` statement to pass:

```javascript
return 1
```

- The second test for the same case where the result will be different which requires one to promote `CONSTANT` to some sort of calculation (variable, formula or function call):

```javascript
return n
```

- The third test for the other case which requires one to write a specific `if (argument == SPECIFIC_VALUE)` with another `return ANOTHER_CONSTANT`:

```javascript
if (n == 7)
  return 42

return n
```

- The fourth test for the same case where the result will be different which requires one to promote `ANOTHER_CONSTANT` to some sort of calculation (variable, formula or function call):

```javascript
if (n == 7)
  return m * 6

return n
```

- The fifth test for the same case where the condition has to be different which requires one to promote specific condition to the proper one:

```javascript
if (n >= 7)
  return m * 6

return n
```

In normal Baby-Steps TDD that would probably have been only 2 or 3 test cases. With Triangulation it is 5 and to make every one of them pass requires a simple transformation of the production code.

Now let's see why these techniques combined make me more productive.

## Willpower Depletion

Did you know that every decision you make costs you some willpower? For example: choosing what to wear in the morning, refusing to eat this tasty cake or to make a design choice in your code. This phenomenon is known as Ego Depletion ([see in wiki](https://en.wikipedia.org/wiki/Ego_depletion)) and it has an experimental evidence. According to this phenomenon self-control and willpower both draw upon a limited pool of mental resources and it can be used up. Usually, these resources are recovered greatly during good night's sleep or slightly after consuming food. Cost per each made decision differs also and even for the same kind of decision can depend on various factors:

- perceived complexity of the problem,
- mood, physical state and perceived fatigue of the person (tired, angry, confused, happy, energized, etc.),
- required effort to make and execute the decision,
- blood glucose levels.

Baby-Steps TDD combined with Triangulation technique optimize for "perceived complexity of the problem" so that every decision is nearly obvious to make and the effort required to make it and execute on it is stupidly small. While this increases the amount of decisions that I need to make it also decreases the complexity and willpower cost of each decision to the point, where after completing the same amount of work I still have plenty of energy and willpower to make any other decisions at work and outside of it.

It is worth noting that these techniques need to be practiced quite a bit to enable this effect - with such incremental design it is important to [avoid getting stuck](/blog/2016/08/30/getting-stuck-while-doing-tdd-part-1-example/). Definitely, try it out and see if it works for you!

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).

## Acknowledgements

Thanks to David Völkel for the great presentation about Baby-Steps TDD. Slides can be found [here](http://www.slideshare.net/davidvoelkel/baby-steps-tdd-approaches).

Thanks to Stephen Guise for the great book ["Mini Habits"](http://minihabits.com/) that has opened my eyes to the reasons why I like these techniques so much and why I love designing software in tiny increments.
