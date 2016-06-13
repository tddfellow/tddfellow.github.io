---
layout: post
title: "TDD as an Enabling Practice or How to be Faster With TDD"
date: 2016-06-13 07:52:56 +0200
comments: true
categories:
- tdd
- skills
- practices
- agile
- xp
- professionalism
- pair-programming
- continuous-integration
- continuous-delivery
- enabling-practices
- exploitative-practices
- dreyfus-model
---

Recently I had a lot of conversations with so many different programmers, with different backgrounds, contexts of work and opinions. What stroke me the most, is that majority feel that TDD is slower than simple automated testing, i.e., `TestFirst` is slower than `TestAfter`.

While digging deeper in their context of work, I could only agree with them: "True, in that context it will be about 50% slower".

Most of the time, though, the context is somewhat looking like this:

- We have some company `ACME`, that does some sort of Agile Software Development (probably Scrum);
- The company `ACME` focuses only on the business parts of Agile Software Development;
- The company `ACME` tried to introduce TDD as a development practice;
- Everything took much longer to be done.

## Enabling practice

Now, let us dive into the development practices of Agile Software Development: development practices are usually coming from Extreme Programming (XP). In XP there are 2 terms: `EnablingPractice` and `ExploitativePractice`.

Exploitative practice gives direct benefit to the team, e.g.: speed boost and quicker feedback from users and stakeholders.

Enabling practice is required for certain other exploitative practice(s) to work.

A good example of exploitative practice is Continuous Delivery. It requires Continuous Integration, Pair-Programming, and Testing to be in place. These 3 are exploitative practices.

## Removing slow practices

Additionally to allowing usage of practices, that make a team go faster and deliver at the higher quality level, enabling practices allow removal of practices, that make a team go slower. For example, Pair-Programming together with TDD allows removal of code review. On most of the teams (especially, of bigger size), this makes for an instant productivity boost.

Pair-Programming, TDD and Continuous Integration, additionally to enabling the team to do Continuous Delivery, also allows replacing feature-branch VCS flow with a trunk-based flow. This allows for smaller iterations and faster user feedback.

## Pair-Programming Done Right

It is worth noting, that removal of code review and introducing of Continuous Delivery is only possible, if Pair-Programming is done right:

- in no case, two beginners should be working in the pair;
- beginners should work together with advanced beginners/competents and proficients/experts;
- advanced beginners/competents should split their time in half between working with beginners and working with proficient/experts.

Terminology `Beginner`, `AdvancedBeginner`, `Competent`, `Proficient` and `Expert` is from Dreyfus Skill Acquisition Model.

That allows for a good trust and mentorship models in your team(s). It enables quick growth and knowledge sharing for every member of the team.

There is another enabling practice which speeds up the knowledge sharing, it is Pair Rotation, that should be done from 1 to 2 times per day, so that for small and middle-sized teams, the bigger the feature is, the higher chance, that everyone on the team have participated in its development, and therefore have enough knowledge about it.

Additionally, Pair Rotation allows for Code Detachment and removal of Code Silos. This in turn, together with TDD, enables Ruthless Refactoring, because you are not afraid:

- to break the code, thanks to TDD,
- to upset an owner of the Code Silo, because there is no owner, thanks to Pair Rotation.

## Bottomline

I think I can go on like this forever, but I believe the idea should be clear. I will be following up with more articles in details on each technique in the future. Stay tuned!

## Thank you for reading!

If you, my dear reader, have any thoughts, questions or arguments about the topic, feel free to reach out to me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you liked my ideas, follow me on twitter, and, even better, provide me with your honest feedback, so that I can improve.
