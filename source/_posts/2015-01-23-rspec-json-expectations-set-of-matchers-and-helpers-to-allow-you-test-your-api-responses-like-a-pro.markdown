---
layout: post
title: "RSpec Json Expectations - Set of matchers and helpers to allow you test your API responses like a pro"
date: 2015-01-23 02:56:19 +0200
comments: true
categories: [ruby, rspec, json, api]
---

[rspec-json_expectations](https://github.com/waterlink/rspec-json_expectations) library provides powerful `include_json` matcher for your RSpec suites. It allows to match string with JSON or already parsed ruby `Hash` against other ruby `Hash`, which is very convenient and creates very readable spec code. Lets jump to some examples.

It can handle some plain json:

``` ruby
it "has basic info about user" do
  expect(response).to include_json(
    id: 25,
    email: "john.smith@example.com",
    name: "John"
  )
end
```

And nested json:

``` ruby
it "has gamification info for user" do
  expect(response).to include_json(
    code: "7wxMw32",
    gamification: {
      rating: 93,
      score: 355
    }
  )
end
```

You can even do some regex matching:

``` ruby
it "has basic info about user" do
  expect(response).to include_json(
    code: /^[a-z0-9]{10}$/,
    url: %r{api/v5/users/[a-z0-9]{10}.json}
  )
end
```

Most can agree, that this method of specifying JSON responses in ruby is very readable, but what about failure messages? How helpful they are?

For example with failure in nested JSON things can become tricky, but this gem solves them quite nice:

``` plain
                 json atom at path "gamification/score" is not equal to expected value:

                   expected: 355
                        got: 397
```

If you match with nested Arrays you will get numbers in your JSON path within failure message, for example:

``` plain
                 json atom at path "results/2/badges/0" is not equal to expected value:

                   expected: "first flight"
                        got: "day & night"

                 json atom at path "results/3" is missing
```

For further reading and instructions: [github](https://github.com/waterlink/rspec-json_expectations) and [cucumber generated documentation](http://www.relishapp.com/waterlink/rspec-json-expectations/docs/json-expectations).

Feedback is highly appreciated, contact me on twitter ([@tdd_fellow](https://twitter.com/tdd_fellow)) or on [github issues](https://github.com/waterlink/rspec-json_expectations/issues).
