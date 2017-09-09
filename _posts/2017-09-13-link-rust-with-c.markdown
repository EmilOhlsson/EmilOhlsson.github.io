---
layout: post
title:  "Link Rust with C libraries"
date:   2017-09-09
author: Emil Ohlsson
categories: rust c
---
One feature I like about Rust is that you can directly call C functions. But
since C requires manually deallocated resources and Rust does this for you there
need to be some kind of glue layer between the two. This is in particular true
if you work on something that allocates resources.

First of all, lets create a Rust project
```
cargo new --bin link-adventures
```
Then change working directory to the rust project directory, `link-adventures`.

In this example, we have a simple C API, which a `_new` function and a
`_destroy` function. The source code resides in `foo.c`
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#include "foo.h"

foo_t *foo_new(const char *msg)
{
    printf("Creating object with msg: %s\n", msg);

    foo_t *foo = malloc(sizeof(foo_t));
    foo->msg = strdup(msg);

    return foo;
}

void foo_destroy(foo_t *foo)
{
    printf("Relaseing object with msg: %s\n", foo->msg);

    free(foo->msg);
    free(foo);
}
```
And the interface, `foo.h`, looks like this
```c
#pragma once

typedef struct {
    char *msg;
} foo_t;

foo_t *foo_new(const char *msg);

void foo_destroy(foo_t *foo);
```
The library can be built using the following commands
```sh
gcc -shared -fPIC -o libfoo.so foo.c
```
We now have a library to work with. Let's start with the rust parts. There,
thankfully, is a utility that helps with automatically creating the rust
interface. This is nice, since writing down the C types in rust code is kind of
cumbersome and boring. The utility, `bindgen` can be installed using
```sh
cargo install bindgen
```
So, using `bindgen` like this
```sh
bindgen -o src/foo.rs foo.h
```
Somewhat abbreviated the created `foo.rs` looks like this:
```rust
#[repr(C)]
#[derive(Debug, Copy)]
pub struct foo_t {
    pub msg: *mut ::std::os::raw::c_char,
}
impl Clone for foo_t {
    fn clone(&self) -> Self { *self }
}
extern "C" {
    pub fn foo_new(msg: *const ::std::os::raw::c_char) -> *mut foo_t;
}
extern "C" {
    pub fn foo_destroy(foo: *mut foo_t);
}
```
As a Rust API, however, this is not very useful. We need to create a mechanism
that automatically calls `foo_destroy`. Depending on the library you are
creating an interface for there might be certain patterns that are more or less
useful. But here the focus is minimal effort. So for my part I will hide the C
API behind a new thin wrapper API.

To do this I will create a new struct, which only stores a pointer to a `foo_t`.
I will also the `Drop` trait, to create a destructor function, and I will lower
the visibility of `foo_t` inside `foo.rs`. So the end result looks like this.

```rust
use std::ffi::CString;

#[repr(C)]
#[derive(Debug)]
struct foo_t {
    pub msg: *mut ::std::os::raw::c_char,
}

extern "C" {
    fn foo_new(msg: *const ::std::os::raw::c_char) -> *mut foo_t;
}

extern "C" {
    fn foo_destroy(foo: *mut foo_t);
}

pub struct Foo {
    ptr: *mut foo_t,
}

impl Drop for Foo {
    fn drop(&mut self) {
        unsafe { foo_destroy(self.ptr as *mut foo_t); }
    }
}

impl Foo {
    pub fn new(msg: String) -> Foo {
        unsafe {
            let foo = foo_new(CString::new(msg).unwrap().as_ptr()) as *mut foo_t;
            Foo{ptr: foo}
        }
    }
}
```
The `Clone` and the `Copy` trait could be removed, as they are not used here.

The next step is to update `src/main.rs` to use this API, and show that the
object is both allocated and deallocated.

```rust
mod foo;

use foo::Foo;

fn main() {
    let foo = Foo::new(String::from("foo-bar"));
}
```

Finally, now that all code is in place, the last step is to make everything
build and in particular link. Since this library is not installed in the system
`cargo` need to be informed on where to find the library.

Rust have made the rather bold move to use Rust as language for the build
script. To add a build script, simply add the file `build.rs`. Build scripts
operate by printing output that is parsed by `cargo`.  A more correct solution
would be to not hard coded values, but to make this as simple as possible that
solution is used.
```rust
fn main() {
    let dir = "/path/to/rust-bindgen";

    println!("cargo:rustc-link-search={}", dir);
    println!("cargo:rustc-flags=-l foo");
}
```
If everything works, then `cargo` should give you a binary to run, but in order
to run it you need to help the linker to find the library.
```
$ cargo build
...
$ LD_LIBRARY_PATH=. cargo run
Creating object with msg: foo-bar
Releasing object with msg: foo-bar
```

That is it, that is what is needed to link with a C application. If you want to
link with a system library then the `build.rs` will be simpler, and might even
be omitted.

To learn more you can read the `cargo` documentation at
[crates.io](http://doc.crates.io/build-script.html)
