---
layout: post
title: "Why are you slow?"
date: 2016-02-05 17:09:47 +0100
comments: true
categories: 
- preaching
- cleancode
- craftsmanship
- professionalism
- software-industry
---

> We are slow because we have the worst codebase!

## So why don't you clean it?

> Because we have to go fast!

I will let you figure out logical inconsistency there.

Ask yourself a question: how much times in your career you were slowed down by
a horrible, dirty, untested code? It doesn't matter who have written it,
usually, it is already there and already a fact you have to deal with.

If your answer was: "Once.. or twice.." (and your career is sufficiently long)
- you can freely stop reading this post and simply be a happy developer.

I was slowed down by bad code horrible amount of times. I come to be sick of
it. I don't want to be slow, because I am trying to go fast. I want my tool to
be clean, I want my creations to be manageable, and I don't want to fear the
code, I have created, or anyone else have created. And I want to go fast and
while enjoying it!

Trust me, I met lots of developers during my career. Not as much as heroes of
our industry have met, but **enough** to be able to tell you, that **every
single** one of them, who has any reasonable length of the career, had same
problems.

And that is totally not normal. Do you understand, that from the outside world,
we, as an industry, are perceived as very non-professional, because of that?
Business even came to a conclusion, that they can't trust us to deliver working
software - and that is how QA role was born.

I will ask again.

## So why don't you clean it?

> My company pays me for features, not for <*insert your statement here*>

That is very common response. And, in essence, it is the truth.

OK, now, let's think. As time passes by, after certain threshold on every
codebase, that is not clean, adding next feature costs more time, even if the
features are of roughly the same size. After 1 year of such development,
features can literally become 2-4 times more expensive. Couple of years in, and
it, practically, becomes impossible to develop anything except "Change the
color of this button" (or provide your own example).

## What does that mean?

One would say: "Features become more expensive over time". Business usually
sees it as: "Arrgh, our programmers slower and slower with each month". So from
the business perspective, after **perceived** effectiveness of developers drops
by 2-4 times, why would business be willing to pay these developers the same
salary? You should be afraid of this question.

## Now imagine land of unicorns!

Now imagine, that you started a bit slower in the beginning (like 8% slower),
but you have kept your effectiveness over time either at roughly same level, or
even have increased it over time. How hard do you think it would be to ask for
a raise, after 2 years of loyal work?

Not hard at all. And probably business will be doing just great (if the
business idea itself was sustainable to begin with, of course), and will be
capable of satisfying the request.

## Why such a thing happens?

Because at the moment we are not perceived as professionals, nor as experts of
our field. We are perceived as just some coding monkeys, who you always need to
ask: "Can this be delivered on Friday" with intimidating tone. And we gladly
reply through our teeth: "I will try", meaning "Just go already"; ending up on
Friday evening with: "I tried" - "You have tried not enough!".

## Clean your code already!

Believe me, investing 15-45 minutes every single day into increasing test code
coverage and some little refactorings will not make you any slower (you
probably already very slow, so that it will not make any perceivable
difference). Rather, over time, you (and your fellow programmers) will start to
actually being bit-by-bit faster, as long as your application (or applications)
get cleaner and cleaner.

It goes without saying, that you should be using proper XP (pair-programming
and TDD) techniques, while writing any new piece of code (read: new class,
module, library, package, etc.). Because it will be extremely easy to unit-test
it with near-100% test coverage. Believe me, that is easy and fun.

Refactoring and covering old and messy parts of an application is not fun,
though. You have to face the truth. Consider it as a chore, like the one you do
to your apartment, when you clean it on a regular basis. And as we all know,
the more you wait to clean an apartment, the harder it would be to do it. And
the function is not linear..

If you have a really big legacy application, you are probably in doing that
sort of thing for 2-3 years, before you can proudly call this application clean
again.

There is one important trick though: prioritize cleaning of parts of
application, that change often. If some part changes once half a year, you
should probably clean it once half a year too.

## You are hired for your expertise!

Believe me, you do have all the knowledge, required to make it.

The only thing that stops you is your inability to say "No", when "No" is the
correct answer from your experience and expertise point of view.

## Parallel example

Do you know that surgeon, before any surgery, washes his hands. You not just
know, it is your expectation! In some countries if he doesn't, he can easily
can be put in jail.

### Do you know, how surgeon washes his hands?

He rubs each finger from 4 different sides 10 times. That is stated in Doctors'
code of ethics. And they have to follow it, since it has power of law.

Why 10 times? Wouldn't 7 be enough, or maybe 14 should be a minimum? It doesn't
matter. It is a discipline, that is to be followed to the letter, and there is
no exceptions.

Well, he probably can rub 11 times, without getting in jail..

Nobody will ever ask surgeon, why he does it. Everyone expects him to do it.

## Back to our universe

You are surgeons!

You are surgeons, that operate on a heart of the business.

With your wrong move, with your mistake, the whole business can go out of
business overnight.

With your wrong move, with your mistake, thousands of people can die (if you
are in certain domains).

So why increasing a chance of your own mistakes (and everybody else on a team)
by not cleaning the code?

Even if you are not in such critical domain, you are part of industry, and you
should be a great professional, just to be an example for others, who might end
up working in such crucial business domain in the future.

And I know, you can pull it off. And you do know it yourself.

So now go, and be professional, be professional for everyone around you.

## Thanks!

This article might be a bit too rough - I believe that is the truth we face
now, as an industry. And let us be the ones fixing it!

I recommend reading this: [Software Engineering Code of Ethics](http://acm.org/about/se-code)
by ACM organization (originally, created in 1999, why are we all still not
using it?!).
