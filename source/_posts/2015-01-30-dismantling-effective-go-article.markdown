---
layout: post
title: "Dismantling Effective Go Article"
date: 2015-01-30 23:39:11 +0200
comments: true
categories: go
---

Go is a nice language and giving tips on how to better write code in that language right away at first language tutorials is awesome.

Article itself: https://golang.org/doc/effective_go.html

And here is a list of my thoughts.

## Formatting

I generally like this approach, it is nice to have such feature in your language tools available right there and with already existing integrations to popular editors.

Except one thing:

> Parentheses
>
> Go needs fewer parentheses than C and Java: control structures (if, for, switch) do not have parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so
>
>     x<<8 + y<<16

So we kill parentheses and replace them with whitespace. I am not entirely sure it is better. How will this work then?

``` go
x << 8 + y << 16
```

And what about this?

``` go
x << 8    +    y  <<  16
```

Probably nobody will write it like that, but one space miss, and lots of minutes lost in debugging.

## Semicolons

Oh, lexer inserting semicolons where it pleases? What can go wrong. Reminds me about javascript and its problems when you omit semicolons.

And this example looked strange to me:

>One consequence of the semicolon insertion rules is that you cannot put the opening brace of a control structure (if, for, switch, or select) on the next line. If you do, a semicolon will be inserted before the brace, which **could cause unwanted effects**. Write them like this
>
>      if i < f() {
>          g()
>      }
>
>not like this
>
>      if i < f()  // wrong!
>      {           // wrong!
>           g()
>      }

First I thought that it will accept this input, put a semicolon after if, compile successfully and always execute statement inside of block, especially because it said "could cause unwanted effects". But turned out this code results in compilation error - so no problem here, but probably this should have been clarified in the article.

## Control Structure

Mandatory block - looks good to me.

But the last example worries me:

>This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in return statements, the resulting code needs no else statements.
>
>     f, err := os.Open(name)
>     if err != nil {
>          return err
>     }
>     d, err := f.Stat()
>     if err != nil {
>          f.Close()
>          return err
>     }
>     codeUsing(f, d)

That code is definitely not narrative, I will put some comments with kind of operation that is done:

``` go
f, err := os.Open(name)   // get input
if err != nil {           // check errors, guard
    return err
}
d, err := f.Stat()        // get additional input
if err != nil {           // check for errors, again, guard
    f.Close()             // cleanups
    return err
}
codeUsing(f, d)           // do something
```

This way it is hard to understand and reason about.

Why not just:

``` go
f = os.Open(name)         // get input
d = f.Stat()              // get input
result = codeUsing(f, d)  // do something
if result.IsErorr() {     // handle errors
    f.Close()             // cleanup
}
```

Of course `f`, `d` and `result` are just `Result`/`Either` monads, they can be either in normal or faulty state. Whenever you try to do anything on `Result` in faulty state, it will just return itself. But `Result` in normal state will proxy call to its contents and wrap it in another `Result` monad (normal or faulty - depends on if call was successful or not). Probably I am too critical about that, because here I used some patterns that are not part of the language itself, but yeah, I can't look at code that mixes guards and actions.

Probably, somebody out there in go-lang world can tell me, is it common to use patterns like `Maybe`, `Result`, `NullObject` and so on in go-lang? Or everybody just go with simple code without any magic behind the scenes, and just do it like in the article's example? Feel free to ping me at twitter [@tdd_fellow](https://twitter.com/tdd_fellow).

## For

Lets move on..

>For strings, the range does more work for you, breaking out individual Unicode code points by parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. (The name (with associated builtin type) rune is Go terminology for a single Unicode code point. See the language specification for details.)

I really like that a term for this "unicode minimal entity" was invented. And after some while the name actually makes a lot of sense. And it sounds good.

## Switch

That surprised me. Feels like assembler:

>Although they are not nearly as common in Go as some other C-like languages, break statements can be used to terminate a switch early. Sometimes, though, it's necessary to break out of a surrounding loop, not the switch, and in Go that can be accomplished by putting a label on the loop and "breaking" to that label. This example shows both uses.
>
>     Loop:
>         for n := 0; n < len(src); n += size {
>             switch {
>                 case src[n] < sizeOne:
>                     if validateOnly {
>                         break
>                     }
>                     size = 1
>                     update(src[n])
>                
>                 case src[n] < sizeTwo:
>                     if n+1 >= len(src) {
>                         err = errShortInput
>                         break Loop
>                     }
>                     if validateOnly {
>                         break
>                     }
>                     size = 2
>                     update(src[n] + src[n+1]<<shift)
>                 }
>             }

Not exactly the original assembler label, one only use it to mark (aka tag) the loop itself and use it to break out of it. Interesting concept, but something clicks in my head when I look at this.

## Type switch

Good one, "Switching on Type", (usually) a code smell, builtin into language. But still it has its own uses when used carefully.

## Multiple return values

Why not tuples? If you just introduce tuples and destructuring, then you don't need multiple return values, only one return value - tuple itself.

## Defer

This is a nice one. Allows to simplify previous example with files even more:

``` go
f = os.Open(name)         // get input
defer f.Close()           // deferred cleanup right after acquiring of `f`
d = f.Stat()              // get input
result = codeUsing(f, d)  // do something
```

Particularly interesting example with trace/untrace follows in the article.

---

To be continued...
