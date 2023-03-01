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
<iframe width="800px" height="200px" src="https://godbolt.org/e?readOnly=true&hideEditorToolbars=true#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGIAMxmpK4AMngMmAByPgBGmMQBQQAOqAqETgwe3r6JpClpjgJhEdEscQmBtpj2hQxCBEzEBFk%2BfpV2mA4Z9Y0ExVGx8bkKDU0tOe2jfeEDZUOBAJS2qF7EyOwc5v7hyN5YANQm/m5ULAQA9GjEmAB0CEfYJhoAgk/PI8ReDvtChwDsVhe%2B2BPwgC3%2BFn2I3QIBASS8BAUEHMZiwVCYXloBAAtA4SCiFkdISY/gARN4g0FoBgjH7mABs4JJkNOBFhSWI4QIyLMZjQSQAnriiMQTABWNwMAlE/7koEgoQQIQMhlMgH7VnszmCHlmFioABumGF%2BIlUt5hP8xLJFIVDP2qCS8SYIqOpIg1NpyrMjIhtspwOh7IRSJR/KFTAUaWA5rMlsBzwDIOuBFWDH2ACoCAg8Aoif7ZQXvfSHU7iC78f53cX7Qo1Qmk4GCDC4SHdfqjdjI9HY/GC5SU2nM9nc/n5cCSXLEyCAH6K%2BtQ5vBxG69Cui0yydvLdW7cvD5fAj7NxcP3j/aeo%2B/PO7l47hNvA/fNxmM/T4HXsevG2314vLn7CwTDhGCb6UkGrYrmGAi0mwcyihuv6Usy/YgieF5cF%2BjbHqeyCvm6i4th2mAelwfbnlu54QfCUG8gwAi4jBR5weU0pISCKHnpSL4XlhjY8XhhxVoRsLER65HvoWd7noOxDphoX6ThwSy0JwYq8H4HBaKQqCcG41jWFCKxrJghxmP4PCkGyWnKUsADWID0mYNxivonCSBpmi8LpHC8AoIAaFZXlLHAsAwIgIArAQNHkJQaAsEkdDxJErAbKoAAc9LYvSkj7MAyDIPsXDOWYvCYPgIp4DCXAyIIIhiOwUi1fIShqF5pC6DVADu5ZJJwPAqWpnk2TpnAAPIIjRDpUPsGVZTleUFUVJX7BAHgJUlxBmRZCy8NZWgLEsCCYEwWAJGCpAOf4fwuW5HAeaQLAgGKgWadpPl%2BQFQU2SF4UQEg8WJfQZAUB6qAbcDIDAMVQRYAaeDrAAangmBdWNTqaZZNBYvE/kQDE7UxOEjQCv1vBE8wxACmNMTaJ01mWfFbCCGNDC0KTI1YEBRjiJzeDXF0Rr%2BSNmCqJ0CIbNpXLVO1tB4DE5ZUx4WDtQQnJPdwyl8AYwAKMjqPo4wZPNfV4hNfwgiKCo6gjZ1%2BiGMYBmWPo8v%2BbAzBsCArKkEaCQcAAnFwB1LI6tTC9i0JuqYljWGYGj7NiY3%2BH51T0xkLgMO4nitHooQzKU5R6Pk6QCOMfg1cXtT9AXQw1R0XQCD0YzZzkdepw3dRTNXgwJHXUxl3oIy9N38ESEsCjGesY93eppBvd5nCzZl2W5flhXFTcr4QLghAkNtZF7cFR0nWdlCDfdvBPS9c/tR9thffttmXQEN2uapHDJzfI134/h13aVX93qcEPj9JYvs0jOEkEAA%3D%3D"></iframe>
There are of course cases where this guideline can be ignored, like for example
if your object is a singleton, or otherwise immovable.

## `constexpr`
Despite popular belief, `constexpr` doesn't mean that the expression will be
evaluated at compile time, it means that result should be available, if needed,
at compile time. So just marking an expression as `constexpr` will not force
compile time evaluation. Jason Turner does a great job at explaining this in his
video on what to use [instead of constexpr], and the long story short is: you
want to use `static constexpr` instead.

## When not to use `static constexpr`
For header constants you want to use `inline constexpr` instead. This will tell
the linker that same definition can exist in multiple files, and will be merged.

[instead of constexpr]: <link to C++ weekly>
[a link]: https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
[implicit rules]: https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html

<!-- vim: set et tw=80: -->
