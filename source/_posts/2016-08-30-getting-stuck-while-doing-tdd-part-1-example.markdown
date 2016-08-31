---
layout: post
title: "Getting Stuck While Doing TDD. Part 1: Example"
description: "Following 3 rules of TDD sounds really simple at first. In practice there is a moment when one has to implement the whole algorithm at once to make currently failing test pass. This is called 'getting stuck' in TDD. In this article we will explore how exactly this happens and how to prevent that."
date: 2016-08-30 15:27:30 +0200
comments: true
categories:
- ruby
- test-driven-development
- tdd
- problem-solving
- triangulation
- getting-stuck-while-doing-tdd
---

Following 3 rules of TDD sounds really simple at first. In practice there is a moment when one has to implement the whole algorithm at once to make currently failing test pass. This is called "getting stuck" in TDD. In this article we will explore how exactly this happens and how to prevent that.

Code examples today will be in Ruby programming language. The technique itself is, of course, language-agnostic.

## TL;DR

"Getting stuck" happens for a couple of reasons:

- wrong order of tests
- production code is not getting more general with each test

This is a series of articles:

1. Part 1: Example (reading this)
2. Part 2: Buggy Code and Forcing Our Way Through (coming soon)
3. Part 3: Triangulation to the Rescue! (coming soon)

## "Getting Stuck" in TDD

Usually "Getting Stuck" follows this pattern:

- write some test and implement it via "simplest thing that might possibly work",
- write another test and implement it again in non-general manner,
- write some more tests in that fashion, while never addressing the fact that production code now looks completely wrong from what it should probably be looking like,
- write new test, that forces us to completely rewrite production code in a complete algorithm just to make it pass.

This last step usually takes minutes to hours depending on the complexity of the problem at hand. Additionally, first few tests are basically wasted time, since they did not produce any bits of knowledge in the production code that persisted in production code in the end. Even worse, there are quite some chances that the algorithm that we have just written is not fully covered by current tests, since we have written it in one go just to make current failing test pass - this is no longer correct TDD and can not guarantee high test coverage, and, therefore, can not guarantee high confidence anymore.

Let's go through a small example on how one can get stuck in TDD:

## Order Kind Validation - Getting Stuck

Let's define the problem at hand first. We have some sort of order request as an input to our system and we need to validate that its kind is correct:

- valid order kinds: `private`, `corporate`, `bundle`,
- order kinds can be combined,
- `private` and `corporate` order kinds can not be combined, otherwise `InvalidOrderError` with message `Order kind can not be 'private' and 'corporate' at the same time`,
- either `private` or `corporate` should be always present, otherwise `InvalidOrderError` with message `Order kind should be 'private' or 'corporate'`,
- if order kind is not in the above list, then we need to raise `InvalidOrderError` with message `Order kind can be one of: 'private', 'corporate', 'bundle'`,
- if order kind is not present or an empty string, then we need to raise `InvalidOrderError` with message `Order kind can not be empty`.

This is fairly simple problem and it is easy to get stuck while doing TDD here. So let's write our first test: "When order has no order_kind, then we should get InvalidOrderError with message 'Order kind can not be empty'":

```ruby
RSpec.describe OrderKindValidator do
  it "fails with message about order kind being empty when it is absent" do
    validator = OrderKindValidator.new

    expect { validator.validate({ items: 42 }) }
        .to raise_error(InvalidOrderError, "Order kind can not be empty")
  end
end
```

And the simplest implementation possible:

```ruby
class OrderKindValidator
  def validate(order)
    raise InvalidOrderError.new("Order kind can not be empty")
  end
end

class InvalidOrderError < StandardError
end
```

Next test is our next simplest edge case - when kind's value is `nil`:

```ruby
it "fails with message about order kind being empty when it is nil" do
  validator = OrderKindValidator.new

  expect { validator.validate({items: 42, kind: nil }) }
      .to raise_error(InvalidOrderError, "Order kind can not be empty")
end
```

It does not fail at all, so we don't have any reason to change the production code. We can already spot a little duplication - `validator` variable. Let's extract it as a named subject of the test suite:

```ruby
subject(:validator) { OrderKindValidator.new }
```

And `OrderKindValidator` can be replaced with `described_class` (RSpec feature), so that we will not have to change too much in case we wanted to change name of the class:

```ruby
subject(:validator) { described_class.new }
```

Next simplest edge case - when kind is an empty array:

```ruby
it "fails with message about order kind being empty when it has zero elements" do
  expect { validator.validate({items: 42, kind: [] }) }
      .to raise_error(InvalidOrderError, "Order kind can not be empty")
end
```

I believe I am spotting annoying pattern now:

```ruby
it "fails with message MESSAGE when it is KIND_CASE" do
  expect { validator.validate({items: 42, kind: KIND_VALUE}) }
    .to raise_error(InvalidOrderError, MESSAGE)
end
```

It would be really nice to write it in this fashion:

```ruby
it_fails_with("Order kind can not be empty").when_order_kind_is_absent
it_fails_with("Order kind can not be empty").when_order_kind_is nil
it_fails_with("Order kind can not be empty").when_order_kind_is []
```

And as another duplication piles up:

```ruby
it_fails_with_order_kind_not_empty = it_fails_with("Order kind can not be empty")

it_fails_with_order_kind_not_empty.when_order_kind_is_absent
it_fails_with_order_kind_not_empty.when_order_kind_is nil
it_fails_with_order_kind_not_empty.when_order_kind_is []
```

Now the next tests look very easy and simple:

```ruby
it_fails_with_order_kind_not_empty.when_order_kind_is [nil]
it_fails_with_order_kind_not_empty.when_order_kind_is [""]
it_fails_with_order_kind_not_empty.when_order_kind_is [nil, nil]
it_fails_with_order_kind_not_empty.when_order_kind_is ["", ""]
it_fails_with_order_kind_not_empty.when_order_kind_is ["private", ""]
it_fails_with_order_kind_not_empty.when_order_kind_is ["private", nil]
```

And they all pass right from the go. The implementation for the `it_fails_with` is looking like this:

```ruby
RSpec.describe OrderKindValidator do
  class ItFailsWith
    def initialize(spec, expected_message)
      @spec = spec
      @expected_message = expected_message
    end

    def when_order_kind_is_absent
      expect_failure("absent", {items: 42})
    end

    def when_order_kind_is(value)
      expect_failure(value.inspect, {items: 42, kind: value})
    end

    private

    def expect_failure(feature, order, expected_message = @expected_message)
      @spec.it("fails with message #{expected_message.inspect} when order kind is #{feature}") do
        expect { validator.validate(order) }
            .to raise_error(InvalidOrderError, expected_message)
      end
    end
  end

  def self.it_fails_with(message)
    ItFailsWith.new(self, message)
  end
end
```

So, let's write our next edge case - when order kind is invalid:

```ruby
it_fails_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    .when_order_kind_is ["invalid"]
```

Pretty neat! And oh, it fails:

```
expected InvalidOrderError with
  "Order kind can be one of: 'private', 'corporate', 'bundle'",
got #<InvalidOrderError: Order kind can not be empty>
```

And the fix:

```ruby
def validate(order)
  if order[:kind] == ["invalid"]
    raise InvalidOrderError.new(
        "Order kind can be one of: 'private', 'corporate', 'bundle'"
    )
  end

  raise InvalidOrderError.new("Order kind can not be empty")
end
```

Let's write our next test - when order kind is `private`:

```ruby
it_does_not_fail.when_order_kind_is ["private"]
```

This fails as expected with `expected no Exception, got #<InvalidOrderError: Order kind can not be empty>`. And to make it pass we need to wrap second `raise` statement in the `if` condition:

```ruby
if order[:kind] != ["private"]
  raise InvalidOrderError.new("Order kind can not be empty")
end
```

The implementation for `it_does_not_fail` looks like that:

```ruby
class ItDoesNotFail
  def initialize(spec)
    @spec = spec
  end

  def when_order_kind_is(value)
    @spec.it("does not fail when order kind is #{value.inspect}") do
      expect { validator.validate({items: 42, kind: value}) }
        .not_to raise_error
    end
  end
end

def self.it_does_not_fail
  ItDoesNotFail.new(self)
end
```

Let's write our next test:

```ruby
it_does_not_fail.when_order_kind_is ["corporate"]
```

And it fails with the expected error: `expected no Exception, got #<InvalidOrderError: Order kind can not be empty>`. The fix is to amend our `if` condition with that case:

```ruby
if order[:kind] != ["private"] && order[:kind] != ["corporate"]
                                # ^  we have added this case  ^
  raise InvalidOrderError.new("Order kind can not be empty")
end
```

And the tests pass. Our next business rule is that one of `private` and `corporate` should be always present:

```ruby
it_fails_with("Order kind should be 'private' or 'corporate'")
    .when_order_kind_is ["bundle"]
```

As expected the test fails:

```
expected InvalidOrderError with
  "Order kind should be 'private' or 'corporate'",
got #<InvalidOrderError: Order kind can not be empty>
```

And to fix it we just need to sprinkle another `if` statement in the middle of the function:

```ruby
if order[:kind] == ["bundle"]
  raise InvalidOrderError.new("Order kind should be 'private' or 'corporate'")
end
```

As expected, the test passes. Now we should test other business rule - order can not be of `private` and `corporate` kind at the same time:

```ruby
it_fails_with("Order kind can not be 'private' and 'corporate' at the same time")
    .when_order_kind_is %w(private corporate)
```

This, as expected, fails with error message:

```
expected InvalidOrderError with
  "Order kind can not be 'private' and 'corporate' at the same time",
got #<InvalidOrderError: Order kind can not be empty>
```

And easiest way to fix that is to add another `if` statement:

```ruby
if order[:kind] == %w(private corporate)
  raise InvalidOrderError.new(
      "Order kind can not be 'private' and 'corporate' at the same time"
  )
end
```

And it passes. Let's test that we can combine `private` or `corporate` with `bundle` order kinds:

```ruby
it_does_not_fail.when_order_kind_is %w(private bundle)
```

And it fails with error: `expected no Exception, got #<InvalidOrderError: Order kind can not be empty>`. To fix this we will have to amend our last `if` condition in the function even more:

```ruby
if order[:kind] != ["private"] && order[:kind] != ["corporate"] &&
    order[:kind] != %w(private bundle)
  # ^    this is our new condition    ^
  raise InvalidOrderError.new("Order kind can not be empty")
end
```

And the test passes. Let's refactor the code a bit:

- First we should extract `order[:kind]` duplication to a local variable `kind`
- Extract common parts of `raise` statement to the private method

After this, `OrderKindValidator` will look a bit cleaner:

```ruby
class OrderKindValidator
  def validate(order)
    kind = order[:kind]

    if kind == ["invalid"]
      fail_with("Order kind can be one of: 'private', 'corporate', 'bundle'")
    end

    if kind == ["bundle"]
      fail_with("Order kind should be 'private' or 'corporate'")
    end

    if kind == %w(private corporate)
      fail_with("Order kind can not be 'private' and 'corporate' at the same time")
    end

    if kind != ["private"] && kind != ["corporate"] &&
        kind != %w(private bundle)
      fail_with("Order kind can not be empty")
    end
  end

  private

  def fail_with(message)
    raise InvalidOrderError.new(message)
  end
end
```

Let's write our next test for the same business rule (now a corporate bundle):

```ruby
it_does_not_fail.when_order_kind_is %w(corporate bundle)
```

And it fails with error: `expected no Exception, got #<InvalidOrderError: Order kind can not be empty>`. To fix this we need to add `&& kind != %w(corporate bundle)` to our last `if` condition again.

Now it seems that we have implemented all the business rules (we have all tests for them). Or did we?

## Bottom Line

Buggy `if`-riddled code is what we've got. We will see why in the next part of "Getting Stuck While Doing TDD" series. Stay tuned!

Today we have implemented our not-so-complex problem at hand while following 3 rules of TDD. The result was not of the best quality and we will take a look why in further articles of these series. You would not want to miss next articles on this tech blog, we still have a lot to talk about:

- Triangulation technique in Test-Driven Development - overlooking this technique might cause one fail at doing TDD (these series),
- Continuous Integration and Continuous Delivery - importance of not impeding others,
- Open-Closed Principle - changing behavior by adding new code,
- Mutational Testing, "Build Your Own Testing Framework" series, Test-Driven Development screencasts and so much more!

## Thanks!

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@waterlink000](https://twitter.com/waterlink000).

If you have any questions or feedback for me, don't hesitate to reach me out on Twitter: [@waterlink000](https://twitter.com/waterlink000).
