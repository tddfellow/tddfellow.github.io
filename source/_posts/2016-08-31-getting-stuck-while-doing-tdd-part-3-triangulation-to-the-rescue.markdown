---
layout: post
title: "Getting stuck while doing TDD. Part 3: Triangulation to the Rescue!"
description: "Following 3 rules of TDD sounds really simple at first. Until you get stuck. Today we are going to learn the Golden Rule of TDD and how to not get stuck while doing TDD."
date: 2016-08-31 02:35:32 +0200
comments: true
categories:
- ruby
- test-driven-development
- tdd
- problem-solving
- triangulation
- getting-stuck-while-doing-tdd
---

Welcome back to the "Getting Stuck While Doing TDD" series. Today we are going to learn the Golden Rule of TDD and how to not get stuck while doing TDD.

## TL;DR

- "As tests get more specific, production code gets more generic".
- `RED` is as important as other in Red-Green-Refactor cycle. If next test does not fail, it is either: already implemented, or has to wait until a later time (until it will fail).
- At its core the Triangulation Technique has the following idea:

  After implementing one business rule (with Red-Green-Refactor) make sure to find all "weirdnesses" or non-generalities in the production code and one-by-one eliminate them by writing a test, that proves such non-generality, and then making it pass while removing non-generality. This is the third cycle of TDD - Mini Cycle.

This is a series of articles:

1. [Part 1: Example](/blog/2016/08/30/getting-stuck-while-doing-tdd-part-1-example/)
2. [Part 2: Buggy Code and Forcing Our Way Through](/blog/2016/08/31/getting-stuck-while-doing-tdd-part-2-buggy-code-and-forcing-our-way-through/)
3. Part 3: Triangulation to the Rescue! (reading this)

Shall we get started?

## Specific/Generic Rule of TDD

> As tests get more specific, production code gets more generic.

When making the next failing test pass, our production code should also pass a whole class of similar tests. Best shown in the very simple example. The task at hand is to write the function `sum(a, b)` that will add two numbers. Let's see us a violation of the Specific/Generic rule:

```ruby
expect(sum(2, 2)).to eq(4)
# => NoMethodError: undefined method `sum'

def sum(a, b); end
# => expected: 4, got: nil

def sum(a, b)
  4
end
# => PASS

expect(sum(2, 3)).to eq(5)
# => expected: 5, got: 4

def sum(a, b)
  if b == 2
    4
  else
    5
  end
end
# => PASS
```

The production code to make this last test pass is as specific as the failing test now. The test of the same class (where we change the value of the `b` parameter) will fail for it:

```ruby
expect(sum(2, 42)).to eq(44)
# => expected: 44, got: 5
```

To follow the Specific/Generic rule we ought to make `4` into `2 + b` like that:

```ruby
def sum(a, b)
  2 + b
end
# => PASS
```

This way, when we change `b` to any value it will still pass the test, aside from the fact, that we didn't do anything about `a`. This is because we still don't have any test showing us, that parameter `a` is important, like the following one:

```ruby
expect(sum(4, 7)).to eq(11)
# => expected: 11, got: 9
```

Again we can make it pass in a very specific fashion by introducing specific `if` statement, or we could do it to pass the whole class of such tests:

```ruby
def sum(a, b)
  a + b
end
# => PASS
```

Have you noticed, that from the test suite side we had to "prove" that some knowledge in the system is important and had to be used? This technique is called Triangulation.

## Triangulation Technique

In the essence, Triangulation technique has a very simple idea at its core:

1. Change certain important* knowledge in the system.
2. Assert that the production code behaves in an accordingly expected manner.

_* - important from the perspective of the system or unit under the test_

## Red-Green-Refactor has to have all stages

One Red-Green-Refactor cycle really has to have all stages in it. And I'm not ranting right now about "Refactor" stage, that is a given. Rather, I insist on the "Red" stage - in TDD, when we write a new test, it has to fail. Writing tests that do not fail is another way to get ourselves stuck while doing TDD. One could ask: "If I can't write this test because it does not fail, what should I do about the requirement it represents?", and the answer is rather simple - either this requirement is already implemented and tested by other tests, or we still need this test and we will get back to it later when it actually will fail.

As we can remember, in the first part of these series, we were going through an `OrderKindValidator` example, and we were writing multiple tests in a row, that were all expecting the same outcome and of course they didn't fail, because we had one line in our function that made them all pass. If we were to sprinkle some other tests, that do fail (like a test for a valid order kind), after making it pass, all of these tests will now be failing and therefore they are good candidates for our next test. Let's see it with our own eyes:

```ruby
it_fails_with("Order kind can not be empty").when_order_kind_is_absent
# => expected InvalidOrderError with "Order kind can not be empty",
# => got #<NoMethodError:
# =>      undefined method `validate' for #<OrderKindValidator:0x000000020335c0>>

class OrderKindValidator
  def validate(order)
  end
end
# => expected InvalidOrderError with "Order kind can not be empty"
# => but nothing was raised

def validate(order)
  raise InvalidOrderError, "Order kind can not be empty"
end
# => PASS
```

Now is the point, where we have to choose our next test, and last time we have chosen the test with the same outcome and it did not go so well. Let's choose a test with different outcome, e.g.: when valid order kind is provided:

```ruby
it_does_not_fail.when_order_kind_is(%w(private))
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can not be empty>
```

Now, we have 2 options, to either check for `order[:kind] == %w(private)` or to check for `order[:kind]` being absent. It does not matter what we choose at this point, so let's go with the first one:

```ruby
def validate(order)
  if order[:kind] == %w(private)
    return
  end

  raise InvalidOrderError, "Order kind can not be empty"
end
# => PASS
```

Now let's apply Triangulation technique. We should always ask ourselves the question: "What is weird about this code?" and "What failing test should I write to point out this weirdness?". First weirdness we can spot is that the validator currently accepts only one order kind - `private`. According to our requirements it should also accept `corporate`:

```ruby
it_does_not_fail.when_order_kind_is(%w(corporate))
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can not be empty>

def validate(order)
  kinds = order[:kind]
  if kinds == %w(private) || kinds == %w(corporate)
    # ...
end
# => PASS
```

We also know, that our system should handle duplicate entries in `order[:kind]`:

```ruby
it_does_not_fail.when_order_kind_is(%w(private private))
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can not be empty>

def validate(order)
  kinds = order[:kind]
  if kinds.include?("private") || kinds == %w(corporate)
    # ...
end
# => expected InvalidOrderError with "Order kind can not be empty",
# => got #<NoMethodError: undefined method `include?'
# => for nil:NilClass>
```

Wow! We, of course, can check for `kinds` to not be `nil`, but I would rather listen to this test failure and put a check for `kinds` being absent (and this makes for our second check, that we could have chosen from):

```ruby
def validate(order)
  kinds = order[:kind]

  if kinds.nil?
    raise InvalidOrderError, "Order kind can not be empty"
  end

  if kinds.include?("private") || kinds == %w(corporate)
    return
  end

  raise InvalidOrderError, "Order kind can not be empty"
end
# => PASS
```

So this passes all our tests. It may look weird, and this is exactly the pointer for us which test to write next to prove, that this weirdness is incorrect:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
  .when_order_kind_is(%w(invalid))
# => expected InvalidOrderError
# => with "Order kind can be one of: 'private', 'corporate', 'bundle'",
# => got #<InvalidOrderError: Order kind can not be empty>

def validate(order)
  # ...

  raise InvalidOrderError,
    "Order kind can be one of: 'private', 'corporate', 'bundle'"
end
# => PASS
```

Production code starts looking not so clean and I think it is time to give things proper names:

```ruby
class OrderKindValidator
  def validate(order)
    kinds = order[:kind]

    if empty?(kinds)
      fail_with("Order kind can not be empty")
    end

    unless valid?(kinds)
      fail_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    end
  end

  def valid?(kinds)
    kinds.include?("private") || kinds == %w(corporate)
  end

  def empty?(kinds)
    kinds.nil?
  end

  def fail_with(message)
    raise InvalidOrderError, message
  end
end
```

There is only one weirdness, that is left for triangulation in current production code, before we can move on to the next requirement - `private` can be duplicated while `corporate` can not:

```ruby
it_does_not_fail.when_order_kind_is(%w(corporate corporate))
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>

def valid?(kinds)
  kinds.include?("private") ||
      kinds.include?("corporate")
end
# => PASS
```

Great, now we can safely go back to our empty order kind edge cases:

```ruby
it_fails_with("Order kind can not be empty").when_order_kind_is([])
# => expected InvalidOrderError with "Order kind can not be empty",
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>

def empty?(kinds)
  kinds.nil? ||
      kinds.empty?
end
# => PASS

it_fails_with("Order kind can not be empty").when_order_kind_is([nil])
# => expected InvalidOrderError with "Order kind can not be empty",
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>

def empty?(kinds)
  kinds.nil? ||
      kinds.empty? ||
      kinds[0].nil?
end
# => PASS

it_fails_with("Order kind can not be empty").when_order_kind_is([""])
# => expected InvalidOrderError with "Order kind can not be empty",
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>

def empty?(kinds)
  kinds.nil? ||
      kinds.empty? ||
      kinds[0].nil? ||
      kinds[0].empty?
end
# => PASS
```

And it is a good opportunity to eliminate some duplication:

```ruby
def empty?(kinds)
  empty_value?(kinds) ||
      empty_value?(kinds[0])
end

def empty_value?(value)
  value.nil? || value.empty?
end
```

Now, it is a good time to triangulate, because we have a weirdness in our code: `kinds[0]`. To prove that this is too specific we can write another test:

```ruby
it_fails_with("Order kind can not be empty").when_order_kind_is(["private", ""])
# => expected InvalidOrderError with "Order kind can not be empty"
# => but nothing was raised

def empty?(kinds)
  empty_value?(kinds) ||
      kinds.any? { |kind| empty_value?(kind) }
end
# => PASS
```

Notice, how every single test that we have written was failing and how easy it was to make it pass. This suggests that we are probably moving in the right direction. Let's test our next requirement - we can combine `private` and `bundle`:

```ruby
it_does_not_fail.when_order_kind_is(%w(private bundle))
# => PASS
```

Wait a minute. This is really bad. We should have a failing test here. This happened because we are checking only for the inclusion of `private` or `corporate` and we do not care about anything else in the `order[:kind]` array. We have to discard this test and try to go with failing version of the same business rule - invalid order kind can not be combined with `private`:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
  .when_order_kind_is(%w(private invalid))
# => expected InvalidOrderError
# => with "Order kind can be one of: 'private', 'corporate', 'bundle'"
# => but nothing was raised

def valid?(kinds)
  return false if kinds[1] == "invalid"

  kinds.include?("private") ||
      kinds.include?("corporate")
end
# => PASS
```

While this works, it leads to two other weirdnesses: `kinds[1]` and `"invalid"`, let's the latter first:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
  .when_order_kind_is(%w(private another_invalid))
# => expected InvalidOrderError
# => with "Order kind can be one of: 'private', 'corporate', 'bundle'"
# => but nothing was raised

def valid?(kinds)
  return false if kinds[1] && kinds[1] != "private"

  kinds.include?("private") ||
      kinds.include?("corporate")
end
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>
# .. and more failures ..
```

Other tests fail now, from them it is possible to see, that second kind should be either `private` or `corporate`:

```ruby
def valid?(kinds)
  return false if kinds[1] &&
      kinds[1] != "private" &&
      kinds[1] != "corporate"

  kinds.include?("private") ||
      kinds.include?("corporate")
end
# => PASS
```

This looks rather clunky, we should make it a bit cleaner:

```ruby
ALLOWED_ORDER_KINDS = %w(private corporate)

def valid?(kinds)
  return false if kinds[1] &&
      !ALLOWED_ORDER_KINDS.include?(kinds[1])

  kinds.include?("private") ||
      kinds.include?("corporate")
end
```

Let's eliminate the other weirdness - `kinds[1]`, it probably should verify all kinds in the array:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
  .when_order_kind_is(%w(invalid private))
# => expected InvalidOrderError
# => with "Order kind can be one of: 'private', 'corporate', 'bundle'"
# => but nothing was raised

def valid?(kinds)
  return false if kinds.any? { |kind|
    !ALLOWED_ORDER_KINDS.include?(kind)
  }

  kinds.include?("private") ||
      kinds.include?("corporate")
end
# => PASS
```

And now this can be greatly simplified by inverting the boolean logic:

```ruby
def valid?(kinds)
  kinds.all? { |kind|
    ALLOWED_ORDER_KINDS.include?(kind)
  }
end
```

Now that we have dealt with all weirdnesses in our production code, let's get back to our requirement:

```ruby
it_does_not_fail.when_order_kind_is(%w(private bundle))
# => expected no Exception,
# => got #<InvalidOrderError: Order kind can be one of: 'private', 'corporate', 'bundle'>
```

Wow! Now it fails exactly as it should. This means that it is now the right time for this test! Let's make it pass by adding `bundle` to the list of allowed order kinds:

```ruby
ALLOWED_ORDER_KINDS = %w(private corporate bundle)
# => PASS
```

Nice! Our next requirement is about `bundle` not being used on its own, i.e.: either `private` or `corporate` is required:

```ruby
it_fails_with("Order kind should be 'private' or 'corporate'")
  .when_order_kind_is(%w(bundle))
# => expected InvalidOrderError with "Order kind should be 'private' or 'corporate'"
# => but nothing was raised

def validate(order)
  # ...

  if kinds == %w(bundle)
    fail_with("Order kind should be 'private' or 'corporate'")
  end
end
# => PASS
```

And this is good enough, because that is really the only case, when this can happen until the list of allowed order kinds is extended by future business requirements. We should at least give this condition a proper name:

```ruby
unless has_required?(kinds)
  fail_with("Order kind should be 'private' or 'corporate'")
end

# ...

def has_required?(kinds)
  kinds != %w(bundle)
end
```

Except, that we could provide duplicated `bundle`:

```ruby
it_fails_with("Order kind should be 'private' or 'corporate'")
  .when_order_kind_is(%w(bundle bundle))
# => expected InvalidOrderError
# => with "Order kind should be 'private' or 'corporate'"
# => but nothing was raised

def has_required?(kinds)
  # Easy to fix if we de-duplicate it with #uniq:
  kinds.uniq != %w(bundle)
end
# => PASS
```

Now it is time to move on to the final requirement about conflicts between `private` and `corporate`:

```ruby
it_fails_with("Order kind can not be 'private' and 'corporate' at the same time")
  .when_order_kind_is(%w(private corporate))
# => expected InvalidOrderError
# => with "Order kind can not be 'private' and 'corporate' at the same time"
# => but nothing was raised

def validate(order)
  # ...

  if kinds == %w(private corporate)
    fail_with("Order kind can not be 'private' and 'corporate' at the same time")
  end
end
# => PASS
```

Of course, `kinds == %w(private corporate)` can be considered too specific for production code, we should triangulate it:

```ruby
it_fails_with("Order kind can not be 'private' and 'corporate' at the same time")
  .when_order_kind_is(%w(corporate private))
# => expected InvalidOrderError
# => with "Order kind can not be 'private' and 'corporate' at the same time"
# => but nothing was raised

if kinds.include?("private") &&
    kinds.include?("corporate")
  fail_with("Order kind can not be 'private' and 'corporate' at the same time")
end
# => PASS
```

And, finally, let's give this condition a proper name:

```ruby
if has_conflicts?(kinds)
  fail_with("Order kind can not be 'private' and 'corporate' at the same time")
end

# ...

def has_conflicts?(kinds)
  kinds.include?("private") &&
    kinds.include?("corporate")
end
```

I believe we are done now. Source for this example can be found in [an open pull request here](https://github.com/waterlink/order_kind_validator/pull/3).

Let's recap how Triangulation technique worked for us here.

## Triangulation Technique in Depth

The main goal of triangulation is to prove that the code is not general enough along some axis (class of tests) by writing a test and then making sure it passes. Effective application of the technique requires to prove and eliminate all such "weirdnesses" or non-generalities from the production code after each Red-Green-Refactor cycle for business requirements. This is, in fact, the 3rd cycle of Test-Driven-Development called Mini Cycle of TDD, it should be executed about every 10 minutes.

Another observation is that following this technique we are introducing only one small piece of knowledge into our production code, for example:

- When writing a test for next business requirement, we are introducing the fact that we need an `if` statement with a certain body (in this example it was a `raise error` statement). Since we can not introduce the `if` statement without a condition we need to put some condition there and we put a very specific condition on purpose since we know that it is tested and it is simple.
- Next, we are proving that this condition is too specific by writing a test, and then making it pass with a more generic solution. This way we are introducing a tiny little bit more knowledge in our production code.
- We are repeating this iterative process until the production code is generic enough for the current specification (test suite). And we start over. This is the Mini Cycle of TDD.

## Bottom Line

Today we have learned the Golden Rule of TDD - "As tests get more specific, production code gets more generic", and we have learned the Triangulation Technique, that allows us to follow this rule in an incremental and confident way. Additionally, we have learned, that following Red-Green-Refactor strictly is important, and this includes even the `RED` stage of this cycle - when the test for business requirement does not fail, it is either: already implemented or it has to wait for later.

This is a series of articles:

1. [Part 1: Example](/blog/2016/08/30/getting-stuck-while-doing-tdd-part-1-example/)
2. [Part 2: Buggy Code and Forcing Our Way Through](/blog/2016/08/31/getting-stuck-while-doing-tdd-part-2-buggy-code-and-forcing-our-way-through/)
3. Part 3: Triangulation to the Rescue! (reading this)

You would not want to miss next articles on this tech blog, we still have a lot to talk about:

- Continuous Integration and Continuous Delivery - importance of not impeding others,
- Open-Closed Principle - changing behavior by adding new code,
- Mutational Testing, "Build Your Own Testing Framework" series, 4 Cycles of TDD, Test-Driven Development screencasts and so much more!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
