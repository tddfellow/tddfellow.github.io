---
layout: post
title: "Using contracts.ruby With RSpec. Part 2"
date: 2015-08-31 18:36:30 +0200
comments: true
categories:
- contracts.ruby
- design-by-contract
- rspec
- ruby
---

Remember [Using contracts.ruby With RSpec](http://waterlink.github.io/blog/2015/04/09/using-contracts-dot-ruby-with-rspec/) ?

RSpec mocks violate all `:class` contracts because `is_a?(ClassName)` returns
`false` for mock. That post describes 2 possible solutions:

- stub `:is_a?`: `allow(my_double).to receive(:is_a?).with(MyClass).and_return(true)`, or
- use `contracts-rspec` gem, that patches `instance_double` RSpec helper.

## Custom validators

Since custom validators have finally landed here:
[egonShiele/contracts.ruby#159](https://github.com/egonSchiele/contracts.ruby/pull/159),
now you can just override `:class` validator to accept all RSpec mocks:

```ruby
# Make contracts accept all RSpec doubles
Contract.override_validator(:class) do |contract|
  lambda do |arg|
    arg.is_a?(RSpec::Mocks::Double) ||
      arg.is_a?(contract)
  end
end
```

Now, RSpec mocks will not violate all the `:class` contracts.

More information can be found here: [Providing your own custom validators](https://github.com/egonSchiele/contracts.ruby/blob/v0.11.0/TUTORIAL.md#providing-your-own-custom-validators).

Additionally this refactoring enabled valuable speed optimization for complex
contracts - validators for them will be evaluated only once and memoized.

## Links

- [Previous part](http://waterlink.github.io/blog/2015/04/09/using-contracts-dot-ruby-with-rspec/)
- contracts.ruby: https://github.com/egonSchiele/contracts.ruby
- contracts.ruby chat: https://gitter.im/egonSchiele/contracts.ruby
- Mentioned PR: https://github.com/egonSchiele/contracts.ruby/pull/159
- [Introduction to contracts.ruby](http://waterlink.github.io/blog/2015/03/05/introduction-to-contracts-dot-ruby/)
- [Great contracts.ruby tutorial](https://egonschiele.github.io/contracts.ruby/)
- [Full fledged documentation PR](https://github.com/egonSchiele/contracts.ruby/pull/195): see here staging docs: https://relishapp.com/contracts-staging/contracts-ruby/docs

## Thanks!

If you have any questions, suggestions or just want to chat about how
contracts.ruby is awesome, you can ping me on twitter
[@tdd_fellow](https://twitter.com/tdd_fellow). If you have any issues using
[contracts.ruby](https://github.com/egonSchiele/contracts.ruby) you can create
issues on corresponding github project. Pull requests are welcome!

Comments on [hackernews](https://news.ycombinator.com/item?id=10147783).

Happy coding! [@tdd_fellow on twitter](https://twitter.com/tdd_fellow).
