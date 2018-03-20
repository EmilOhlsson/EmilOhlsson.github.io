---
layout: post
title:  "Makefile tutorial"
date:   2018-03-20
author: Emil Ohlsson
categories: Makefile c
---
For some reason it seem that Makefiles has gotten the reputation that they are
hard to use and learn, almost to the point that people just seem scared even to
deal the slightest with them. Like it's some kind of crazy magic of the old
ones. At least in the linux world there are a lot of other build systems, where
actually many of more popular acutally generate Makefiles. In my experience this
is a bit harder to troubleshoot, as you both need to troubleshoot the system
that generated the Makefile, and the generated Makefile itself. So having some
knowledge on how Makefiles work is really handy.

Many of the tutorials I've read go to great lengths on drawning dependency
graphs etc to explain Makefiles. For me that just didn't do it, so I hope that
this is description that would have helped me getting started.

This text will look at _C_ projects, but most of this can be translated to
pretty much any input that need some kind of processing to create some kind of
output. I will start with explicit Makefiles, and work my way through what
`make` knwo and can help you with

Makefiles track changes in input files and use that information to build new
output files when the input is newer than the input. So for a C program, the
listing would be something like this:
```makefile
program: program.o library.o
program.o: program.c
library.o: library.c
```

`program` is built by linking `program.o` and `library.o`. And these two object
files are built from their respective C source code. Already at this level
`make` can draw two important conclusions: If I only change `library.c` then
there is no need to recompile `program.c`, and there is no dependency between
`library.o` and `program.o` which means they can be built in parallell.

`make` need know how to convert input files to ouput files, even though `make`
already know how to do this for a lot of file types as you'll see later. For the
example above it would be something like
```makefile
program: program.o library.o
	gcc -o program program.o library.o

program.o: program.c
	gcc -c program.c

library.o: library.c
	gcc -c library.c
```

As you can see the commands needed to convert the input source files to target
built files are listed indented below the dependency listing. This need to be a
tab, `make` is really picky about this.

Even though this works, you can see that this is very error prone and sensitive
to typos, which leads us to...

## Variables
Variables can be used to make the Makefile easier to maintain, so say if we want
to be able to easily change the compiler, then we would need to write something
like this
```makefile
CC = gcc

program: program.o library.o
	$(CC) -o program program.o library.o

program.o: program.c
	$(CC) -c program.c

library.o: library.c
	$(CC) -c library.c
```

Variables are expanded with the syntax `$(VARIABLE)`. The `CC` variable used
here is commonly used to indicate the C compiler. Other commonly used variables
are `CFLAGS` for flags to the compiler, `CXX` for the C++ compiler etc.

There are also a bunch of variables that are called automatic variables.
Automatic variables are variables that expand to meaningful information in the
the instruction part of the makefile, which is also known as recipe. The
automatic variables are a bit cryptic in their naming, as they are called things
like `$@` and `$^`, which are also about the two automatic variables that you
really need to care about. The manual has a full list of [automatic variables]
for reference.

The `$@` variable expands to the target file, and `$^` expands to all the input
files. So the example above can now be shortened to
```makefile
CC = gcc

program: program.o library.o
	$(CC) -o $@ $^

program.o: program.c
	$(CC) -c $^

library.o: library.c
	$(CC) -c $^
```

## Making the Makefile shorter
There are a couple of steps that can be used to `make` the above build script
shorter: First it can be noted that both compile recipes are the same, and they
will be for all C files. So by using the wildcard, `%`, this can be written as
```makefile
program: program.o library.o
	$(CC) -o $@ $^

%.o: %.c
	$(CC) -c $^
```

As it happens, `make` actually alread knows how to create object files from C
source code.  `make` has a list of built in implicit rules. For example, `make`
knows that C files should be compiled with
```makefile
	$(CC) $(CPPFLAGS) $(CFLAGS) -c %^
```
 `make` also know that rograms should be linked with
```makefile
	$(CC) $(LDFLAGS) %^ $(LDLIBS) -o %@
```
This might be a bit much to take in att once, but what this means is that as
long as you use the variables listed above, then you actually don't need to
write any recipes for C programs. As long as you use `CFLAGS` for C flags,
`CPPFLAGS` for C Preprocessor flags, `LDFLAGS` for linked flags, and `LDLIBS`
for libraries. So the example makefile can actually be shortend to
```makefile
CC = gcc

program: program.o library.o
```

One quirk to take note of here is that the name of the output file must be the
same as one of the input files, in this case `program`. Just as with automatic
variables the is a catalogue of [implicit rules] in the manual for reference.

### A tiny special case
If you only need to build `program` from `program.c` you actually don't need any
Makefile at all, you can just type `make program`, which will expand to a
compilation and a linking of `program.c` to `program`

## Beyond the basics
The steps listed above are pretty much all you need for a really basic build
script. You can probably leave this text now, and start using Make.

However, for more complete build scripts you likely need code for cleaning up
built files. But, as you might realize, removing a file does not result in an
output file. To be able to support this Make uses something called _phony
targets_. Phony targets are targets that does not result in actual files, but
you want to run for some kind of side effect.

Common phony targets include `all`, `clean` and `install`. `all` is
often the first target in the makefile, which is also the target `make` defaults
to. This is often used to build everything.

To clean up built files the target `clean` is often used, and for installing the
target `install` is often used.

Adding these targets to the exmple script would look something like this
```makefile
.PHONY: all clean

CC = gcc

all: program
clean:
	rm -rf *.o

program: program.o library.o
```

## Common structuring and final notes
There are probably as many structures of Makefiles as there are developers, and
many Makefiles can be really hard to decipher. For my part I've found that each
target directory that should contain the built files is usually the best place
to place your Makefiles. For the most simple case this may be the same directory
as the source code. For the more complex scenarios this can be a separate
directory.

For more complex build systems, where different targets have different ouput
directories it might be worth spending some time to write some common code that
can be included using the `include` directive.

If you are using `gcc`, then it might be worth knowing that `gcc` can create
files for dependency tracking, which allow `make` to rebuild files if there
included header files have changed. This is something that is hard to do in
normal makefiles. To generate this kind of ouput you need to give `gcc` the
`-MMD` flag, and then use
```makefile
-include *.d
```

This tells `make` that include all `.d` files, but if the files does not exist,
then it is not an error.

Hope this text was useful for you, and good luck with your Makefiles.

## Common conventions
 * To support cross compilation of your code it is common to use the variable
   `CROSS_COMPILE` like this `CC = $(CROSS_COMPILE)gcc`
 * `DESTDIR` and `PREFIX` are used to control where you install files, and they
   can be used like `INSTALLDIR $(DESTDIR)/$(PREFIX)/bin`, where `PREFIX` can be
   set using `PREFIX ?= usr`
 * Use `all` as the first target, which does a full build.

[automatic variables]: https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
[implicit rules]: https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html
