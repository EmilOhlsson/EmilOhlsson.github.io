---
layout: post
title:  "Makefile tutorial"
date:   2018-03-20
author: Emil Ohlsson
categories: Makefile c
---

* What does const, constexpr, constinit , consteval mean?
* `constexpr` might not actually behave like you think
* When to use `static constexpr` (almost always)
* When not to use `static constexpr`
* Assumptions you can make in constexpr function (static_assert in input)

* `static constexpr` class members, doesn't affect size
* `constexpr const`?
* `constinit`
* `consteval`

Just as Jason turner has pointed out in his excellent video on what to use in
video on what to use [instead of constexpr], for a lot of people, `constexpr`
doesn't mean what you think it means. Just to get it out of the way, I don't
blame them. How `const` things work in C++ is confusing. There are, as of C++20,
at least 5 different ways of making `const` things in C++. There is the "good
old" `const`, there is `define`, and there is also `constexpr`, `constinit` and
`consteval`. All of these things imply different things, and it can be hard to
keep track of what all of these mean

This post is an attempt at trying to break down what they all mean, and to give
some form of recommendations on how to use them.

## A simple `const`
First, the most simple one is `const`. This is pretty straightforward, and it
has been around a long time. It simply means: _this will not change_, and
actually changing it will be undefined behavior. Making things `const` is
usually a good habit, as it potentially allows the compiler to better optimize
your code. It also prevents you from accidentally changing something you
shouldn't.

### When not to `const`
There is really only one case where you don't want to use `const`, and that is
related to `class` members. Making something `const` might force the compiler to
copy objects instead of moving them.
https://godbolt.org/z/xb13Y3s5c should give a reasonable example

There are of course cases where this guideline can be ignored, like for example
if your object is a singleton, or otherwise immovable.

## `constexpr`

[instead of constexpr]: <link to C++ weekly>
[a link]: https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
[implicit rules]: https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html

<!-- vim: set et tw=80: -->
