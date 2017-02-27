---
layout: post
title:  "Generator macros patterns in C"
date:   2017-02-26 21:45:46 +0100
categories: c macros
---
Usually I don't like to use macros in C. Mostly because it's fairly easy to
obfuscate what the code does, and I would argue that the reason for choosing C
is to be able to roughly what each line translates to. However, macros also
allow you to implement solutions that are otherwise hard to do, or very error
prone.

A very common problem I encounter in C is the problem of translating `enum`
values to strings. It can be for debugging purposes to represent error codes in
a readable way. To implement this I often use something that I think is called a
generator pattern. It looks like this
``` c
#define SYMBOL_MAP(G) G(FOO) G(BAR) G(BAZ)
#define SYMBOL_ENUM(s) s,
#define SYMBOL_STR(s) #s,

typedef enum {
    SYMBOL_MAP(SYMBOL_ENUM)
} symbol_e;

const char *names[] = {SYMBOL_MAP(SYMBOL_STR)};
```
This might look a bit scary, but it's not that bad when you get use to it.
First, the `SYMBOL_MAP`, takes a macro, and applies it to a list of symbols. In
this case, _FOO_, _BAR_, _BAZ_. Next, `SYMBOL_ENUM` and `SYMBOL_STR` is defined
as macros that translates their argument to _something followed by a comma_, and
_something translated to a string followed by a comma_. So, when passing
`SYMBOL_ENUM` or `SYMBOL_STR` to `SYMBOL_MAP` the macros get expanded to
something usable when declaring an `enum` or a list of strings.

Now, this might not seem like much, but the upside is that you only need to list
your symbols once. And, you know that both the list of strings, and the order of
the `enum` symbols are the same. So you can safely convert the symbol `FOO` to
the string `"FOO"` using `names[FOO]`. This is useful to make the code more
readable, as long as you don't lose your mind over the macros.
<!-- vim: set tw=80 et ts=4 ss=4 sw=4 : -->
