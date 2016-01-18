---
layout: post
title: "Introduction to contracts.ruby"
date: 2015-03-06 00:55:40 +0200
comments: true
categories:
  - ruby
  - design by contract
---

## Slides from my talk on RUG-B Mar 2015

A short introduction to a powerful Design by Contract technique and its implementation in ruby contracts.ruby.

Design by Contract allows one to do defensive programming in very elegant fashion, allows to set contracts on methods (expectations on input - arguments; and on output - return result) and invariants on classes. This allows to reason about code much much better.

<iframe src="//www.slideshare.net/slideshow/embed_code/45498085" width="476" height="400" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe>

## Classical defensive programming

Lets start from simple code example:

```ruby
def add(a, b)
  a + b
end
```

If you want to be really confident in implementation and usage of this method, you would probably use something like that:

```ruby
def add(a, b)
  raise "a should be Fixnum or Float" unless a.is_a?(Fixnum) ||
    a.is_a?(Float)
  raise "b should be Fixnum or Float" unless b.is_a?(Fixnum) ||
    b.is_a?(Float)
  result = a + b
  raise "result should be Fixnum or Float" unless result.is_a?(Fixnum) ||
    result.is_a?(Float)
  result
end
```

Which definitely provides guarantees for input and output values.

But this code is extremely ugly, unmaintainable and unreadable. You can always extract `assert`-like helper methods, but it will not improve readability too much, you want to have just this simple `a + b` in the body of this method.

## `gem "contracts"`

```ruby
Contract Num, Num => Num
def add(a, b)
  a + b
end
```

This code does the same thing, but readability at a totally different level. Developers who know haskell may find this notation quite familiar.

## Design by contract

When applying design by contract technique to development of any system or service, it allows you to answer the following questions:

- What does it expect? - Restrictions on input data for the system.
- What does it guarantee? - Restrictions on output data (return value) of the system.
- What does it maintain? - Restrictions on the inner state of the system (if your system is stateful, of course).

## Benefits

Benefits of being able to answer this questions and enforce them on a runtime level are:

- Clients of your system can be confident using its public APIs. They can be sure, that if they provide something wrong, then they will get a convenient error immediately. And they can be sure, that system returns the right value as a result.
- System or service itself can be confident in its own operations. Implementation of system, that is covered with contracts, can assume that all the data flowing through the system is right and expected, and don't waste time (and lines of code, and sanity of the developer/maintainer) on different checks, conversions and so on (ie on defensive programming), it can just do what it needs to do, in confident, concise and convenient way, right up to the point.

## `assert` on steroids. And it is not only about types

Up until now it may seem like some kind of runtime type-checking system. But it is not, it is way more powerful.

You can check for exact value:

```ruby
Contract 200, nil, :get => "ok"
```

You can check for types:

```ruby
Contract User, Time => Or[TrueClass, FalseClass]
```

You can check for anything that is available to you at runtime:

```ruby
Contract ActiveUser => Rating
def rating_for(active_user)
  # .. calculate rating for active user ..
end

class ActiveUser
  def self.valid?(user)
    user.last_activity > 2.weeks.ago
  end
end
```

As you expect when contract check on `active_user` argument happens, it will just call `ActiveUser.valid?(active_user)` and in case of falsy result will raise contract violation error.

## Very useful contract violation errors

```ruby
ContractError: Contract violation for argument 1 of 1:
    Expected: ActiveUser,
    Actual: #<User:0x00000101059540> {last_activity=27.11.2014}
    Value guarded in: Object::rating_for
    With Contract: ActiveUser => Rating
    At: (irb):10
    ... backtrace ...
```

This kind of errors tell you, what exactly you did wrong and where exactly you did it wrong. It is totally different from usual `NoMethodError :something for nil:NilClass`, because usually these kind of no-method errors can occur in totally different part of codebase comparing to where these errors actually were introduced. Contract violation will be issued exactly at the place where you passed invalid data into or out from your system. So that when you see a contract violation error, there is a high chance that you already know how to fix it.

## Pattern matching, sorta..

You can say even method overloading. Very simple example:

```ruby
# factorial in classic way
def factorial(n)
  if n == 1
    1
  else
    n * factorial(n - 1)
  end
end
```

```ruby
# factorial using pattern matching
Contract 1 => 1
def factorial(_)
  1
end

Contract Num => Num
def factorial(number)
  number * factorial(number - 1)
end
```

When I saw this example, my first reaction was: "Wow!". I was very excited about this feature.

## Something useful with pattern matching

Last example was not particularly useful for our everyday development, but here you go.

Imagine you have a concurrent evented system, that needs to make asynchronous requests to some external http service(s). You may eventually end up with handler functions like these:

```ruby
# Classical way
def handle_response(status, response)
  if status == 200
    transform_response(JSON.parse(response))
  else
    wrap_in_error(status, response)
  end
end
```

```ruby
# And using pattern matching:
Contract 200, JsonString => JsonString
def handle_response(status, response)
  transform_response(JSON.parse(response))
end

Contract Fixnum, String => JsonString
def handle_response(status, response)
  wrap_in_error(status, response)
end
```

## Limitless benefits

- All your input data is consistent
- All data flows inside of your system are consistent
- State of your system is consistent
- Output of your system is consistent (or it is a contract violation error)
- Blows up loudly on any logical error in your system

Last point is extremely important, because sometimes logical errors in classical programs will not lead to any failure at all, they will just do the wrong thing. For example, transfer money to wrong bank account. In such mission critical systems it is really important to fail fast to not allow error to propagate throughout your system.

## Caveats: Performance

|Benchmark|Slowdown|
|:---|:-----------|
|`a+b`|900% slowdown|
|production system with network IO|5-10% slowdown|
|`NO_CONTRACTS=1`|0% slowdown|

<p/>

First benchmark is simple comparision of `a + b` with and without contract. Since `a + b` itself is very fast, then the slowdown is huge. But if you try to benchmark any real world system, that actually does something useful (communicates to other services through network for example), then slowdown is very very small.

And you have ability to disable contracts in production with `NO_CONTRACTS=1` environment variable. But beware, you lose extremely important benefit of blowing up on logical error immediately before letting error propagate. This benefit itself outweights these 5-10%, at least for me.

## Useful links

- [Github](https://github.com/egonSchiele/contracts.ruby)
- [Nice tutorial](http://egonschiele.github.io/contracts.ruby)
- [Creator](https://github.com/egonSchiele)
- [Me, co-maintainer](https://github.com/waterlink)

If you have any questions or suggestions, you can always reach me out on twitter [@waterlink000](https://twitter.com/waterlink000). If you have any issues with using `contracts.ruby`, you can always create an [issue on github](https://github.com/egonSchiele/contracts.ruby/issues) and Pull Requests are welcome.

<a href="http://www.codeproject.com" rel="tag" style="display:none;">CodeProject</a>
