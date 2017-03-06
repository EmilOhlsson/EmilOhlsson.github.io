---
layout: post
title:  "Reading lines in rust"
date:   2017-03-06 20:57:46 +0100
author: Emil Ohlsson
categories: rust
---
I recently tried out an online coding game called [CodinGame][codingame]. It's
very similar to games like project euler, hackerrank, and advent of code. The
main difference is that you have your program compete against other peoples
programs.  It is also more graphical than most other similar games, which also
makes it really fun to watch.

As I usually do for coding in my spare time I decided to go for Rust. For people
not familiar with Rust, it is a system programming language with very
interesting memory features. This allows you to write memory safe code, without
using a garbage collector. For me this is a very attractive feature in a
language.

For this CodinGame site you can choose language, and it will provide you with
the boiler plate code needed for reading input from `stdin` and parse it into
integers etc, so you can focus on the main problem.

This is all very good, but I feel that they could have written in i little bit
cleaner fashion. The code seem to be auto generated, but I think that even that
could be somewhat better.

Their suggestion to start is
```rust
fn main() {
    let mut input_line = String::new();
    io::stdin().read_line(&mut input_line).unwrap();
    let n = parse_input!(input_line, i32);
    let mut input_line = String::new();
    io::stdin().read_line(&mut input_line).unwrap();
    let q = parse_input!(input_line, i32);
}
```
For my part, unless fast implementation is part of the problem I have started to
rewrite it as this:
```rust
fn get_line() -> String {
    let mut buffer = String::new();
    io::stdin().read_line(&mut buffer).unwrap();
    return buffer.trim().to_string();
}

fn get_spliteline() -> Vec<String> {
    let line = get_line();
    line.split_whitespace().map(|t: &str| t.to_string()).collect::<Vec<String>>()
}

fn main() {
    let n = parse_input!(get_line(), usize); 
    let q = parse_input!(get_line(), usize); 
}
```
Perhaps it's just a matter of taste, but I prefer to have my code split up into
readable chunks.

Anyway, this aside, [CodinGame][codingame] is a fantastic game and you should
really try it out.

[codingame]: https://www.codingame.com
<!-- vim: set tw=80 et ts=4 ss=4 sw=4 : -->
