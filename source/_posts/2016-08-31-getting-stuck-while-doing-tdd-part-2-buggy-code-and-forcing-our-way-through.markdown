---
layout: post
title: "Getting stuck while doing TDD. Part 2: Buggy Code and Forcing Our Way Through"
description: "Following 3 rules of TDD sounds really simple at first. Until you get stuck. Today we are going to see the results of getting stuck while doing TDD and scratch the surface of how to avoid this outcome."
date: 2016-08-31 02:35:06 +0200
comments: true
categories:
- ruby
- test-driven-development
- tdd
- problem-solving
- triangulation
- getting-stuck-while-doing-tdd
---

Welcome back to the "Getting Stuck While Doing TDD" series. Today we are going to see the results of getting stuck while doing TDD and scratch the surface of how to avoid this outcome.

Code examples today will be in Ruby programming language. The technique itself is, of course, language-agnostic.

## TL;DR

- It is painful and difficult to force your way through when getting stuck in TDD.
- It results in degraded guarantees from TDD (such as test coverage, semantical stability, and confidence).

Ways to avoid this outcome:

- do not write tests that will not fail with the current production code
- choose next test to write that will address particular detail about production code that is wrong or not general enough (Triangulation)

Finally, do not forget to remove redundant tests if any.

This is a series of articles:

1. [Part 1: Example](/blog/2016/08/30/getting-stuck-while-doing-tdd-part-1-example/)
2. Part 2: Buggy Code and Forcing Our Way Through (reading this)
3. Part 3: Triangulation to the Rescue! (coming soon)

## Buggy if-riddled code

Buggy `if`-riddled code is what we've got. It is even not so easy to read. While we can refactor it to be more readable that won't change the presence of bugs, though. Let's still do it to understand what happens in this code better:

```ruby
class OrderKindValidator
  def validate(order)
    kinds = order[:kind]

    validate_only_known(kinds)
    validate_has_required(kinds)
    validate_no_conflicting(kinds)
    validate_non_empty(kinds)
  end

  private

  def validate_non_empty(kinds)
    if empty?(kinds)
      fail_with("Order kind can not be empty")
    end
  end

  def validate_no_conflicting(kinds)
    if has_conflicting(kinds)
      fail_with("Order kind can not be 'private' and 'corporate' at the same time")
    end
  end

  def validate_has_required(kinds)
    if has_no_required(kinds)
      fail_with("Order kind should be 'private' or 'corporate'")
    end
  end

  def validate_only_known(kinds)
    if invalid?(kinds)
      fail_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    end
  end

  def empty?(kind)
    kind != ["private"] && kind != ["corporate"] &&
        kind != %w(private bundle) &&
        kind != %w(corporate bundle)
  end

  def has_conflicting(kind)
    kind == %w(private corporate)
  end

  def has_no_required(kind)
    kind == ["bundle"]
  end

  def invalid?(kind)
    kind == ["invalid"]
  end

  def fail_with(message)
    raise InvalidOrderError.new(message)
  end
end
```

Structure of the class, actually, sounds just right, but conditions are not good:

```ruby
def empty?(kind)
  kind != ["private"] && kind != ["corporate"] &&
      kind != %w(private bundle) &&
      kind != %w(corporate bundle)
end
```

Really? It does not do what it says. At all. It basically just solves the problem very specifically to the tests. I can easily come up with a test that will break it:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    .when_order_kind_is ["almost anything"]

# Error: expected InvalidOrderError with
#    "Order kind can be one of: 'private', 'corporate', 'bundle'",
# got #<InvalidOrderError: Order kind can not be empty>

# or other test
it_does_not_fail.when_order_kind_is %w(corporate corporate)
```

```ruby
def has_conflicting(kind)
  kind == %w(private corporate)
end
```

This at least does what it says. But only for one specific case, instead of general one. One test that I can come up with right away:

```ruby
it_fails_with("Order kind can not be 'private' and 'corporate' at the same time")
    .when_order_kind_is %w(private corporate bundle)

# and another one:
it_fails_with("Order kind can not be 'private' and 'corporate' at the same time")
    .when_order_kind_is %w(corporate private)
```

```ruby
def has_no_required(kind)
  kind == ["bundle"]
end
```

While this may work for our current requirements, it is really confusing for the reader. Method name says: "has no required kind" while method body checks if it is only `bundle`. And it does not work well with that edge case:

```ruby
it_fails_with("Order kind should be 'private' or 'corporate'")
    .when_order_kind_is %w(bundle bundle)
```

While this case is quite unlikely, nothing in business rules forbid that and some other part of the system may as well duplicate `bundle` kind for some reason or it may be a user input mistake.

```ruby
def invalid?(kind)
  kind == ["invalid"]
end
```

This method, indeed, checks that `kind` is `invalid`. Literally `"invalid"`. Which would mean, that all kinds except exactly `"invalid"` are allowed. This is not true according to our business rules. In fact, we have already written the failing test for this some moments ago:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    .when_order_kind_is ["almost anything"]
```

Let's comment out these failing tests and try to force-TDD our way through these bugs by uncommenting and fixing them one-by-one following Red-Green-Refactor loop:

## Forcing our way through

So, let's uncomment our first failing test:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    .when_order_kind_is ["almost anything"]
```

We are expecting `validate_only_known` to fail with its message and that means `invalid?(kinds)` should return true. To make it return `true` in this case and preserve its old behavior we will need to remove `private`, `corporate` and `bundle` from `kinds` and check that it is not empty:

```ruby
def invalid?(kinds)
  (kinds - %w(private corporate bundle)).any?
end
```

See how we had to write the whole thing in one go. There is no chance to write it incrementally because there will be a bunch of tests that fail. Wait! While it does not fail for any tests related to invalid kinds, it fails for all tests related to emptiness:

```
OrderKindValidator
  fails with message "Order kind can not be empty"
    when order kind is [""] (FAILED - 1)
    when order kind is ["", ""] (FAILED - 2)
    when order kind is ["private", ""] (FAILED - 3)
    when order kind is absent (FAILED - 4)
    when order kind is nil (FAILED - 5)
```

So we need to change more production code to make this one tiny test pass. It looks like `validate_non_empty` is a culprit now - it is being called after `validate_only_known`. It should be the other way around:

```ruby
def validate(order)
  kinds = order[:kind]

  validate_non_empty(kinds)
# ^ we moved this up here ^

  validate_only_known(kinds)
  validate_has_required(kinds)
  validate_no_conflicting(kinds)
end
```

Oh! Now a bunch of other tests fails:

```
OrderKindValidator
  fails with message "Order kind should be 'private' or 'corporate'"
    when order kind is ["bundle"] (FAILED - 1)

  fails with a message "Order kind can not be 'private' and 'corporate' at the same time"
    when order kind is ["private", "corporate"] (FAILED - 4)

  fails with a message "Order kind can be one of: 'private', 'corporate', 'bundle'"
    when order kind is ["almost anything"] (FAILED - 2)
    when order kind is ["invalid"] (FAILED - 3)
```

From failure messages it is possible to guess, that the culprit is `empty?(kinds)` function that fails in too much cases now, such as: `["bundle"]`, `["private", "corporate"]`, `["almost anything"]` and `["invalid"]`. This is because it was not doing what it said it was:

```ruby
def empty?(kinds)
  kinds != ["private"] && kinds != ["corporate"] &&
      kinds != %w(private bundle) &&
      kinds != %w(corporate bundle)
end
```

And this is why it was hard to change the order of validations. We will have to completely rewrite this function. Let's start small and see which tests fail:

```ruby
def empty?(kinds)
  false
end
```

The failures are:

```
OrderKindValidator
  fails with message "Order kind can not be empty"
    when order kind is ["private", ""] (FAILED - 1)
    when order kind is [nil] (FAILED - 2)
    when order kind is ["", ""] (FAILED - 3)
    when order kind is nil (FAILED - 4)
    when order kind is [""] (FAILED - 5)
    when order kind is absent (FAILED - 6)
    when order kind is [nil, nil] (FAILED - 7)
    when order kind is ["private", nil] (FAILED - 8)
    when order kind is [] (FAILED - 9)
```

Good, only tests related directly to this case are failing. So one-by-one we can construct our condition while fixing these test failures:

1. `kinds.nil?`
2. `|| kinds.empty?`
3. `|| kinds[0].nil?`   (turned out to be redundant in the end)
4. `|| kinds[0].empty?` (turned out to be redundant in the end)
5. `|| kinds.any? { |k| k.nil? || k.empty? }`

After refactoring `empty?` the function now is looking this way:

```ruby
def empty?(kinds)
  absent_or_empty?(kinds) ||
      kinds.any? { |kind| absent_or_empty?(kind) }
end

def absent_or_empty?(value)
  value.nil? || value.empty?
end
```

And all tests, finally, pass. It took a lot of effort and re-writing to get this one little test to pass. This is what we call "Getting Stuck" in TDD. There is always an order of tests that will lead to this result almost for any somewhat complex problem.

The code can be found in GitHub repository in [an open pull request here](https://github.com/waterlink/order_kind_validator/pull/2/files).

Almost guaranteed ways to get stuck in TDD:

- write tests that do not fail,
- do not address weird results of "simplest thing that could possibly work" to make the test pass and moving on to the next business rule,
- make production code a mirror of the tests and too specific, not general.

And to not get stuck is to do the opposite:

- do not write the test that will not fail (wait until later, when it will fail), and
- always first write the test that will point out next deficiency in the current production code (in TDD this is called Triangulation), and
- while making some failing test pass, make sure that the change in production code covers not only this one specific test, rather, a whole class of tests (Golden Rule of TDD: As tests get more specific, production code gets more generic).

## Bottom Line

Today we have seen how bad the results of getting stuck while doing TDD can be. In the next article of these series, we will explore Golden Rule of TDD and the technique called Triangulation, that allows us to incrementally test-drive code in a way, that it will always be conforming to the Golden Rule of TDD and therefore will never get us stuck. Stay tuned!

You would not want to miss next articles on this tech blog, we still have a lot to talk about:

- Triangulation technique in Test-Driven Development - overlooking this technique might cause one fail at doing TDD (these series),
- Continuous Integration and Continuous Delivery - importance of not impeding others,
- Open-Closed Principle - changing behavior by adding new code,
- Mutational Testing, "Build Your Own Testing Framework" series, Test-Driven Development screencasts and so much more!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
