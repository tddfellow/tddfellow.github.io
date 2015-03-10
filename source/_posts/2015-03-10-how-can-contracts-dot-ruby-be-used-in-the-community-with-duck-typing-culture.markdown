---
layout: post
title: "How can contracts.ruby be used in the community with duck typing culture?"
date: 2015-03-10 11:11:23 +0100
comments: true
categories:
- ruby
- contracts.ruby
- design-by-contract
---

So, given simple example:

```ruby
Contract Num, Num => Num
def add(a, b)
  a + b
end
```

One can ask: "But it is ruby, what about duck typing, I want just pass two things that have certain methods defined on them"

And my answer, you can easily do that:

```ruby
Contract RespondTo[:save, :has_valid?], RespondTo[:to_s] => Any
def assign_user_a_default_email(user, default_email)
  user.email = default_email unless user.has_valid?(:email)
  user.save
end
```

This is a built-in `RespondTo` contract. You can get a list of all of them here: http://egonschiele.github.io/contracts.ruby/#built-in-contracts

If you have any questions, you can always ping me at twitter [@waterlink000](https://twitter.com/waterlink000)
