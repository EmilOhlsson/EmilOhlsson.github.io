---
layout: post
title:  "Use cookiecutter to stub projects"
date:   2024-10-02
author: Emil Ohlsson
categories: misc
---
I tend to start a lot of small projects, and for some newer programming
languages there are tools for quickly getting started with a new project. In
particular Rust has `cargo` which is really convenient for getting started. With
a simple call to `cargo new` you will have a project directory set up, complete
with a hello world program and everything you need to start building your
program.

For C and C++ there is such standard way you should set up your code. It all
depends on what tools you intend to use, how you want to structure your code and
what kind of system you will target.

So, recently I've stumbled on [Cookiecutter][cookiecutter] which is a tool for
setting up project directories based on templates. It's based on python, and it
is quick and easy to set up a new template for your particular need. I did a
quick experiment, and after some quick troubleshooting I had something up and
running

## A simple example
First, make sure `cookiecutter` is installed using
```sh
pip install --user cookiecutter
```
Next, I set up the following structure
{% raw %}
```
makefile-template
├── cookiecutter.json
└── {{cookiecutter.project_slug}}
    ├── {{cookiecutter.project_slug}}.cpp
    └── Makefile
```
where `{{cookiecutter.project_slug}}.cpp` is a simple _hello, world_ program,
and `Makefile` has the following content:
```Makefile
.PHONY: all
all: {{cookiecutter.project_slug}}
```
Text inside the double curly braces will be replaced by cookiecutter.

Next, taking a look at `cookiecutter.json` will show what parameters we're
working with:
```json
{
    "project_name": "Simple C++ project",
    "project_slug": "{{cookiecutter.project_name.lower().replace(' ', '_').replace('+', 'p')}}"
}
```
{% endraw %}
These paramters have associated default values, and these will be filled in the
user when setting up the new project directory. So as you can see
`project_slug`, which was used above in the example directory is listed in the
`json` file, along with a default value based on the project name.

Finally, to demonstrate hwo this can be used; simply run
```sh
cookiecutter path/to/makefile-template
```
And you will be prompted with
```
  [1/2] project_name (Simple C++ project): my_hello
  [2/2] project_slug (my_hello):
```
and you will now see a new directory called `my_hello`:
```
my_hello
├── Makefile
└── my_hello.cpp
```

Cookiecutter seem very capable, and there are a bunch of ready made templates
available online already. There are also several ways to store templates; public
git repositories, zip files, directories etc. So do take a look, it's worth
looking closer at.

[cookiecutter]: https://cookiecutter.readthedocs.io/en/latest/index.html

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
