---
layout: post
title: "Why do you need to be careful with loop variable in Go"
date: 2016-01-08 02:28:45 +0100
comments: true
categories: 
- golang
- concurrency
- programming
- computers
- pitfalls
---

## Taking reference of loop variable

Lets start off with simple code example:

```go
for message := range inbox {
        outbox <- EnhancedMessage{
                // .. more fields here ..
                Original: &message,
        }
}
```

Looks quite legit. In practice it will often cause race conditions. Because
`message` variable is defined once and then mutated in each loop iteration. And
we are giving pointer to this variable away to some concurrent collaborator.
This causes race condition and very confusing bugs.

Above code can be re-written as:

```go
// local scope begins here
        var (
                message Message
                ok bool
        )
        for {
                message, ok = <-inbox
                if !ok {
                        break
                }

                outbox <- EnhancedMessage{
                        // .. more fields here ..
                        Original: &message,
                }
        }
// local scope ends here
```

Looking at this code, it is quite obvious why it would have race conditions.

Correct way of doing that would be to either define a new variable manually
each iteration and copy `message`'s value into it:

```go
for message := range inbox {
        m := message
        outbox <- EnhancedMessage{
                // ...
                Original: &m,
        }
}
```

Another way of doing that would be to take control of how loop variable works
yourself:

```go
for {
        message, ok := <-inbox
        if !ok {
                break
        }

        outbox <- EnhancedMessage{
                // ...
                Original: &message,
        }
}
```

Taking into account, that until the `EnhancedMessage` is processed by
concurrent collaborator and garbage collected, variables, created during each
iteration, i.e.: `m` and `message` for both examples, will stay in memory.
Therefore it is possible to just use pass-by-value instead of
pass-by-reference to achieve the same result. It is simpler too:

```go
for message := range inbox {
        outbox <- EnhancedMessage{
                // ...

                // Given the fact that `EnhancedMessage.Original` definition
                // changed to be of value type `Message`
                Original: message,
        }
}
```

Personally, I prefer latter. If you know of any drawbacks of this approach
comparing to other 2, or if you know of entirely better way of doing that,
please let me know.

## Running goroutine, that uses loop variable

Example code:

```go
for message := range inbox {
        go func() {
                // .. do something important with message ..
        }()
}
```

This code might look legit too. You might think it will process whole inbox
concurrently, but most probably it will process only a couple of last elements
multiple times.

If you rewrite the loop in a similar fashion as in previous section, you would
notice that `message` would be mutated while these goroutines are still
processing it. This will cause confusing race conditions.

Correct way of doing that is:

```go
for message := range inbox {
        go func(message Message) {
                // .. do something important with message ..
        }(message)
}
```

In this case, it is basically the same as copying the value to the new defined
variable at each iteration of the loop. It just looks nicer.

## Thanks!

If you have any questions, suggestions or just want to chat about the topic,
you can ping me on twitter [@waterlink000](https://twitter.com/waterlink000) or
drop a comment on [hackernews](https://news.ycombinator.com/item?id=10864593).

Especially, if you think I am wrong somewhere in this article, please tell me,
I will only be happy to learn and iterate over this article to improve it.

Happy coding!
