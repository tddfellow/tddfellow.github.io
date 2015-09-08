---
layout: post
title: "Intention-revealing code"
date: 2015-09-08 21:24:52 +0200
comments: true
categories: 
- programming
- pragmatic
- rant
- golang
- computers
- design
---

Lets start off with very simple code:

```go
func toJSON(post Post) string {
        return fmt.Sprintf(
                `{"post_title": "%s", "post_content": "%s"}`,
                post.Title,
                post.Content,
        )
}
```

This is very simple code and it is easy to understand what it is doing and what
it is doing wrong:

- It tries to marshal `post` struct to custom `JSON` representation.
- It fails when there are special characters in these strings.
- It does not use standard `MarshalJSON` interface.

It can be fixed in a pretty simple way:

```go
func (post Post) MarshalJSON() ([]byte, error) {
        return json.Marshal(map[string]interface{}{
                "post_title":   post.Title,
                "post_content": post.Content,
        })
}
```

And at the usage site, you can now just use standard `encoding/json` package
capabilities:

```go
rawPost, err := json.Marshal(post)
if err != nil {
  return err
}

// do stuff with rawPost
```

And now you can notice that tests do not pass. And the place that failed is
totally unrelevant.  Long story short. Name of the original method was not
revealing any real intent: it was actually specific json representation for
usage with external API, but normal `json.Marshal` is used by this same
application for providing responses to its own HTTP clients.

Were the name a bit more intention-revealing, nobody would waste their time
on finding this out by trial and mistake:

```go
// should probably even sit in different package
func marshalToExternalAPIFormat(post Post) ([]byte, err) {
        // ...
}
```

And this is only a tip of the iceberg of how non-intention-revealing code can
trip you over.
