---
layout: post
title: "Using contracts.ruby with RSpec"
date: 2015-04-09 22:51:53 +0300
comments: true
categories:
- contracts.ruby
- design-by-contract
- ruby
- rspec
---

## Issues with RSpec mocks

Lets start from example:

```ruby
class Example
  include Contracts

  Contract Something => Any
  def do_something(something)
    something.call
  end
end
```

And its corresponding spec:

```ruby
require "rspec"

RSpec.describe Example do
  let(:example) { Example.new }
  let(:something) { instance_double(Something, call: :hello) }

  it "works" do
    expect(example.do_something(something))
      .to eq(:hello)
  end
end
```

Pretty straightforward unit test for `Example#do_something`. But if you run `rspec` you will get:

```
ContractError: Contract violation for argument 1 of 1:
    Expected: Something,
    Actual: #<RSpec::Mocks::InstanceVerifyingDouble:0xa401a0 @name="Something (instance)">
    Value guarded in: Example::do_something
    With Contract: Something => Any
```

It happens because class contracts use `#is_a?` to determine if contract matches or not. Simply: `something.is_a?(Something)` is required to be `true`.

But if we try to do it with `instance_double`, that is what we get:

```ruby
require "rspec/mocks/standalone"
something = instance_double(Something)
something.is_a?(Something)                              #=> false
something.is_a?(RSpec::Mocks::InstanceVerifyingDouble)  #=> true
```

## Solution to problem

Pretty straightforward one:

```ruby
let(:something) { instance_double(Something, call: :hello) }

before do
  allow(something)
    .to receive(:is_a?)
    .with(Something)
    .and_return(true)
end
```

But this can be boring to type it each time you need an `instance_double` while working with `contracts`. So here you go:

```ruby
# Gemfile
group :test do
  gem "contracts-rspec"
end
```

Run `bundle` to install `contracts-rspec` gem.

```ruby
# your spec file
require "contracts/rspec"

RSpec.describe Example do
  include Contracts::RSpec::Mocks

  # .. write code as in first example ..
end
```

Now you are covered. Inclusion of `Contracts::RSpec::Mocks` slightly alters behavior of `instance_double`. Now it automatically stubs out `#is_a?(Klass)` to return `true` on the class it was created from. In our case `Something`. This happens here: https://github.com/waterlink/contracts-rspec/blob/master/lib/contracts/rspec/mocks.rb#L4-L8

You can include it only in contexts you need, or you can do it globally from `spec_helper` like you do usually include spec helpers.

## Links

- Gem: https://github.com/waterlink/contracts-rspec
- contracts.ruby: https://github.com/egonSchiele/contracts.ruby
- contracts.ruby chat: https://gitter.im/egonSchiele/contracts.ruby
- Born from here: [egonSchiele/contracts.ruby#14](https://github.com/egonSchiele/contracts.ruby/issues/14)
- [Introduction to contracts.ruby](http://waterlink.github.io/blog/2015/03/06/introduction-to-contracts-dot-ruby/)
- [Great contracts.ruby tutorial](https://egonschiele.github.io/contracts.ruby/)

## Thanks!

If you have any questions, suggestions or just want to chat about how contracts.ruby is awesome, you can ping me on twitter [@waterlink000](https://twitter.com/waterlink000). If you have any issues using [contracts.ruby](https://github.com/egonSchiele/contracts.ruby) or [contracts-rspec](https://github.com/waterlink/contracts-rspec) you can create issues on corresponding github project. Pull requests are welcome!
