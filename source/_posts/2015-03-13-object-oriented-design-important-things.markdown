---
layout: post
title: "Object Oriented Design. Important things"
date: 2015-03-13 00:23:45 +0200
comments: true
categories:
- ruby
- oop
- design
- tdd
- actor-model
- dependency-inversion
- dependency-injection
---

## Slides from my talk [@brainly](https://twitter.com/BrainlyGroup) 12 Mar 2015

*Disclaimer: This is my personal vision, based on my knowledge and experience. If you want to challenge it, ask questions, provide feedback and discuss, feel free to ping me on twitter: [@waterlink000](https://twitter.com/waterlink000), I would love to hear from you.*

Code examples are in ruby/pseudo-code.

<iframe src="//www.slideshare.net/slideshow/embed_code/45775223" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

## Object Oriented Programming

Most notable features of OOP:

- Encapsulation - keep data together with behavior that needs that data, effectively hiding this data from everything else.
- Inheritance - usually a subclassing, inheriting all behavior and data, in some language even all private details.
- Polymorphism - ability to substitute instances of one class with instances of others.

### How important these features are for OO design?

Imagine, that given 100 points you want to distribute them between these 3 features, and each number will represent an importance of corresponding feature.

This would be my answer (and my personal opinion):

|Encapsulation|Polymorphism|Inheritance|
|:--:|:--:|:--:|
|80|40|-20|

And 100 = 80 + 40 + (-20) :)

### Why inheritance is so bad?

Because it increases coupling (usually): subclass (usually) depends on some data and/or behavior of its superclass. Even worse: (usually) it is not public data/behavior. I.e.: it violates principle of encapsulation badly. (Usually).

There are actually cases, when you do want your classes to be in inheritance hierarchy, it is the case, when your domain has the same hierarchy naturally in real world. And type inheritance doesn't really mean behavior inheritance.

### So how to avoid violation of encapsulation using inheritance?

- Be careful, use only public interfaces of your superclass.
- Replace inheritance with composition and delegation, because when you use only public interfaces of superclass, then you don't really need inheritance.

## Coupling

### Why coupling is bad?

Imagine you need to change behavior of class X.

You will have to change any other piece of code base, that directly depends on this behavior of class X.

More coupling you have, more changes will have to take place, and probably, in totally unrelated parts of codebase.

As a result it:

- Exponentially increases time required for change
- Invites bugs (lots of)

### Dependency

Dependency, is basically what coupling is, - is not really your friend, so you need to watch out for them.

### How to deal with dependencies?

- Dependency inversion principle - Instead of referring foreign system/package/class/module/whatever, refer an abstract interface.
- Dependency injection - Technique, that allows to provision all dependencies to parts of your system and basically fulfill all required interfaces.

### An example

```ruby
class Answer
  def rating
    RatingService.new.rating_for(comments, upvotes, downvotes)
  end
end
```

What is the problem with this code? - It is a dependency `Answer -> RatingService`. What if you wanted to A/B test different rating models? You will definitely have troubles with that approach, especially if it is not the only place, that reference `RatingService`.

### Use dependency injection!

```ruby
class Answer
  def initialize(rating_service)
    @rating_service = rating_service
  end

  def rating
    @rating_service.rating_for(comments, upvotes, downvotes)
  end
end
```

Now you can easily have multiple rating services and A/B test them, or do whatever you want, it is really flexible.

### It is still coupling

But coupling to abstract interface, instead of real implementation, which means, you can exchange different implementations without changing users of this interface. Which basically improves polymorphism features of your code. It is very loose coupling.

## Test Driven Development

### How is it related to OO design?

It provides very short feedback on your OO design:

- If you have troubles writing test - your design is wrong and you need to step back
- If you don't like how your test look like - your design is wrong and you need to step back
- If you have troubles making test green - your design is wrong and you need to step back

It is really almost like pairing partner if used right! Of course pairing still provides even better feedback loop - real-time continuous feedback loop!

### Unit tests vs integration tests

Integration tests are scam!:

- Very slow => bad feedback loop
- Exponential count of paths to test

### Unit tests just don't work. Are they?

Everything works in isolation != the whole system works as expected.

Testing in isolation = providing fake objects and/or mocks for all your dependencies

Mocks and fake objects can make your unit test green, but in fact the code is broken, because one of the dependencies has changed its behavior or even public interface in unexpected fashion.

### Answer: Cut your system at value boundaries!

Instead of method call boundaries. And we arrive at Actor Model.

## Actor Model

Given this example:

```ruby
class RatingService
  def rating_for(comments, upvotes, downvotes)
    # .. calculate rating somehow ..
  end
end
```

Problems with this class:

- Any user of this class will have to stub out its `#rating_for` in unit tests.
- It invites additional behavior to be added (since it is as simple as adding additional public method), which will kill single responsibility feature of this class.

### Actor Model solves this

Warning, it is ruby pseudo-code:

```ruby
actor RatingService
  comments, upvotes, downvotes, outbox = inbox.fetch
  # .. calculate rating somehow ..
  outbox.send(rating)
end

rating_service = RatingService.new
rating_service.start
```

### Way better now:

- Allows unit testing easily without mocks and fake objects: by peeking inside its inbox in unit tests instead, and checking that it received right message.
- It is really hard to pack more responsibility to this, since it has no methods, it has just one body, that is responsible for processing exactly one message.

*You of course can encode something strange in message, and organize your own method dispatch mechanism through actors inbox, but that is just silly (usually).*

### Easy to unit test:

```ruby
# creating actor, but not starting it
rating_actor = RatingCalculator.new

# dependency injection of rating actor
answer_actor = Answer.new(answer_id, rating_actor)
answer_actor.start

render_actor = Render.new(answer_actor)
render_actor.run
expect(rating_actor.inbox)
  .to include([comments, upvotes, downvotes, outbox])

# fake response from rating actor
outbox.send(3.5)
expect(render_actor).to render_rating(3.5)
```

In unit tests you start only one actor - actor under test, and all other actors just get instantiated correctly and passed in as a dependencies where needed. Since they are not started, they will not consume any messages from their inbox, which means that you can consume these inboxes from your unit test, and check that the messages that arrived at inboxes are expected.

## When the code is done

- It works!
- It is readable (future me will not curse me for writing this code)
- It has no duplication
- And it is as short as possible (while maintaining all of the above)

### Code comments

Basically a code smell (I'm not talking about documentation comments)

Example:

```ruby
# when user is active
if activity_service.has_events(user.id, min_date: 2.weeks.ago)
  && !user.fraud?
```

Which is bad from more than one point of view, it literally should have been:

```ruby
if user.active?
# or
if activity_service.user_is_active?(user)
# .. more variations can be here ..
# .. but all of them will be better ..
```

### If code needs comment:

- it is not readable
- it fails to communicate its intent

You should be able to read the code and understand it. In that order: read -> understand.

You shouldn't interpret it in your head. You shouldn't have Ruby (or your favorite language here) instance running in your head.

## To sum it up

- Inheritance is good only in very rare cases
- Coupling and dependencies are not your friends, take them under control with dependency inversion & injection
- TDD as a shortest feedback cycle for your OO design
- Write code in such way, that you would thank yourself for that in the future

Recommended reading: "Pragmatic Programmer: From Journeyman to Master" by Andrew Hunt and David Thomas. It is insanely good, concise book with lots of awesome references to other resources.

## Thanks!

If you have any questions or suggestions, you can always reach me out on twitter [@waterlink000](https://twitter.com/waterlink000).
