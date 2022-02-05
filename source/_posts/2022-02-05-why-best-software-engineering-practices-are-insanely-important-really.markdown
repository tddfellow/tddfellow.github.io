---
layout: post
title: "Why Best software Engineering Practices Are Insanely Important, Really"
date: 2022-02-05 14:43:04 +0200
comments: true
categories:
---

"So, what software engineering practices do you like to use?" — I asked another developer the other day. And the answer has wholly shaken me:

"What do you mean by "software engineering practices?" — was the developer's response.

When I explain what I mean by these three words (apparently, I took for granted), the person starts listing technologies. This conversation has been repeating a bit too frequently for my liking recently.

Look, I get it. Technologies knowledge and experience are vital for a software engineer. However, the fundamental practices of software engineering are insanely important nevertheless.

Let's discover why.

<!-- more -->

## Holy Grail of Software Development

Why do these practices exist in the first place, and why are countless books written on them? And why is the industry trying to improve on the existing ones and invent new ones?

Why just a bunch of technology skills is not enough?

That's because we all want to get stuff done fast. And we want it done with a high level of quality. And we want it to be valuable.

The "Holy Grail of Software Development" is the ability to deliver fast and with high quality.

While innovative programming language or framework can improve speed and reduce the chance of making mistakes, the improvement effect is negligible in "speed x quality x value."

What matters then?

How the software is developed creates a tremendous effect on speed and quality.

It's the processes applied, techniques used, disciplines practiced.

## Whether You Know it or Not, You're Still Using Practices.

Just not necessarily the good ones.

Slap some code together, use a lot of copy-paste from StackOverflow and your previous project, and get it to production as quickly as possible? — That's a software engineering practice.

It gives you great speed (at first) and, of course, reduces quality substantially. Do that for a few weeks, and you'll inevitably slow down because you'll constantly be debugging all the bugs produced.

Then, on the other hand, there is another practice of unit-testing and integration-testing the code and refactoring it until it becomes "Clean Code." — This is a simple example of a few good practices applied:

- Automated testing.
- Refactoring.
- Clean Code.

## Not Every "Best" Practice is Always Good — Context is King

Let's say that you've committed to using a whole bunch of best practices, such as keeping your code decoupled and flexible using design patterns. Moreover, you've decided to exhaustively implement and test all the different scenarios that you think could happen in the future.

These are all that many would consider best practices.

That sounds quite reasonable, however, applied in the wrong context, that could lead to the following:

- The business never leveraged that extra flexibility because no need in the future came to pass.
- You still had to maintain more code that was more complex to support these flexibilities for many years. That's an extra cost — WASTE.
- It took you originally longer to implement the functionality — WASTE.
- Oh, and half of the scenarios you came up with also never happened. The feature changed significantly after customer feedback, so most cases became impossible later on. — WASTE.
- Additionally, one of the flexibilities constrained the implementation of another flexibility that became required later on. So the business had to go for a trade-off and deliver less value to the customer. — ACTUAL BUSINESS VALUE DAMAGE.

As you can see, the idea was to increase quality, increase speed in the future, and deliver as much customer and business value as possible.

What happened instead?

The quality was excellent — that's all good!

Now, the speed was initially slower, and it kept being slower in the future because none of the expected came to pass.

The best thing you can expect in software development is change (not the one you expected). Also, prepare to have your expectations and assumptions hammered all the time.

Finally, too much flexibility harmed the business value.

The same set of practices could've been fantastic in another context, where you, for example, precisely know what will happen in the future. An example scenario is when you're rebuilding an existing successful system.

In that case, you would have gotten lots of speed, quality, and business value throughout the endeavor.

## Software Engineering Practices Are Based on Principles

What are these practices based on? How to detect whether the context is proper or not? When to apply and when not? How to apply?

You guide the choice of practices and patterns by the software engineering principles!

When you adopt multiple principles (such as YAGNI, DRY, SOLID, etc.), they will guide you when you need to act in specific ways (e.g., apply one practice or another, or make this or that choice).

The challenge would be when multiple principles you've adopted conflict with each other.

That's when you'll have to make trade-offs.

## Where do Principles Come From? How to choose?

Principles should be based on values. The values that you, your team, and your organization adopts.

For example, when I was talking about the "Holy Grail of SWE," I've mentioned the "speed x quality x value" as the three most important ones. That's a value statement.

Also, who said that these three values are the silver bullet to success? Nobody knows. Every person and organization can have its own set of values.

However, for any organization or group to achieve a big goal and execute a complex strategy, it would need to align everybody involved on the same set of values.

Once you and your group have agreed on what values you should adopt to achieve the challenging long-term goals, you must pick or create the principles that will get you there.

Picking existing principles that are tried and tested to improve software engineering in terms of the values you've chosen is the most straightforward approach. However, that shouldn't deter you from thinking about a custom set of principles that would work for your group's values in particular.

## Example Values That I Like to Optimize For

I'll give you a few values that I usually go for in most software products where the only constant is the change itself:

- Shortest lead time — how long does it take from idea inception to value delivery to the hands of the user.
- Lowest inventory — how much work is "done" but isn't being used by anybody (by users, other developers, or other team members).
- Highest quality — how many bugs or failures the software has. It should behave precisely as the development team expects it to.
- Largest valuable impact — how valuable is the software to the user, customer, and the business.

Once these values are defined, it's pretty simple to pick the principles to guide the decisions.

## Example Software Engineering Principles

- LEAN product and development
- YAGNI/KISS
- Evolutionary Design
- Simple Design
- SOLID
- High-coverage automated testing (no manual QA)
- No throwing responsibilities over to another department — Extreme ownership
- Code stewardship (next level after ownership)

And given the principles, you can pick or design your software engineering practices.

## Example Software Engineering Practices

- TDD & BDD & ATDD
- Continuous Refactoring
- CI/CD
- Trunk-Based Development (TBD) & "Live on Main"
- Clean Code
- Slicing work items as small as possible
- Instant Code Review (aka Pair-programming)
- Ensemble programming
- Rolling pair rotation
- Spike & PoC (for experimentation)
- Design Patterns

Most of these are parts of original Extreme Programming, and this is no surprise because the values I have listed above are almost the same as values of XP.

Moreover, there are a few more principles and practices that are modern, and they could be considered as the evolution of XP.

## Values, Principles & Practices Are Technology-Agnostic

Now I want to return to that situation where the developer responded with a list of technologies when asked about practices.

Here's the thing: the practices, principles, and values are universal. No matter which technology you use, you can rely on your practices, given that your principles and values haven't changed.

Moreover, when you master the practices and principles, you'll be able to change technologies and tools like gloves while maintaining high parameters on all your values and delivering excellent results!

## Conclusion

Of course, if you are starting in software engineering, knowing **specific technologies** is most essential for you.

That's because your effectiveness is a product of your tools experience and the practices you apply. And to use and understand the practices, you need to have at least a bit of skill with the programming language and basic problem-solving.

However, beyond that, the equation:

TOOLS SKILL x PRACTICES SKILL x SOFT SKILLS

kicks in!

So the most effective way to develop as a software engineer is to have both a robust set of hard and soft skills and learn and apply the best software engineering practices (that are applicable in your context).

Thank you for reading!

If you have other ideas on the topic, [feel free to send me a Tweet](https://twitter.com/tdd_fellow)!
