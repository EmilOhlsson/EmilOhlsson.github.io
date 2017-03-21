---
layout: post
title:  "Creative error handling in C"
date:   2017-03-21
author: Emil Ohlsson
categories: c macros
---
Error handling in C is always something that have bothered me a bit. For code
that handles allocated resources it is particularly hard. If you don't have any
allocated resources a good approach is usually something in the lines of the
example below.

```c
error_e function(void *param)
{
    int *val = param;

    if (!param) return ERROR_PARAMETER;

    for (int i = 0; i < 10; i++) {
        if (i == *p) return ERROR_FOO;
    }

    return ERROR_OK;
}
```

For code where you don't have anything allocated it is usually best to only do
an early return. Perhaps log some error, but not much more. But as soon as you
start to handling allocated resources, such as semaphores and memory, this
quickly become bothersome.

```c
error_e function(stuff_t *handle)
{
    if (!handle) return ERROR_PARAMETER;

    if (pthread_mutex_lock(handle->mutex) == 0) {
        int *val = malloc(sizeof(int));
        if (!val) {
            if (pthread_mutex_unlock(handle->mutex)) {
                return ERROR_SAD_STATE;
            } else {
                return ERROR_MEMORY;
            }
        }
        /* Do stuff */
        free(val);
        if (pthread_mutex_unlock(handle->mutex)) {
            return ERROR_WEIRD_STATE;
        } else {
            return ERROR_OK;
        }
    }
    return ERROR_LOCK;
}
```

Even with two resources this become rather clunky to work with. And consider if
there is some feature addition which requires an additional resource to be
allocated. That would be really nasty to work with. There are ways to work
around this though.

A very simple solution is to use `goto`. A lot of people consider `goto` to be
bad practice. My view is that if it make the code more readable, then by all
means use `goto`. If it makes the code less readable, use something else. A
common usage for `goto` is something like the example below.

```c
error_e function(stuff_t *handle)
{
    error_e error;
    bool locked = false;
    int *val = NULL;

    if (!handle) return ERROR_PARAMETER;

    if (pthread_mutex_lock(handle->mutex)) {
        error = ERROR_LOCK;
        goto on_error;
    }
    locked = true;

    val = malloc(sizeof(int));
    if (!val) {
        error = ERROR_MEMORY;
        goto on_error;
    }

    /* Do stuff */

    error = ERROR_OK;
on_error:
    if (locked) {
        pthread_mutex_unlock(handle->mutex);
        error = ERROR_WEIRD_STATE;
    }
    free(val);
    return error;
}
```

Might be a bit more code, but it is easier to read, and easier to spot missing
resource deallocations. It is also less nested.

A _gcc_ extension allows the programmer to register a cleanup function for a
variable, which can look kind of weird. But it allows the compiler to deallocate
resources for you.

```c
static void my_free(int **p)
{
    free(*p);
}

void function(void)
{
    int __attribute__((cleanup(cleanup))) *val = malloc(sizeof(int));
}
```

Of course you can hide this monstrosity behind some kind of macro. For example
the Jansson JSON C library probably does something like this, and I've seen
other code bases use this approach as well. If you are not used to it then it is
a bit scary, and you might be tempted to clean up yourself, which will lead to
double free. But Apart from that, then it can be very handy.

Finally, I've been tinkering a but with a macro monstrosity that is loosely
based on the `goto` approach. But by using macros it is possible to add a
relatively light weight stack trace, which can be very handy when tracking down
a bug. All you need is a function that takes a stack, and pushes an entry to it,
and a macro that creates entry information for you.

So, by using the following macro:

```c
#define ERR_GOTO_NEW(handler, var, code)                   \
    do {                                                   \
        var = err_push(err_new(code), __FILE__, __LINE__); \
        goto handler;                                      \
    } while (0)

#define ERR_GOTO(handler, var)             \
    do {                                   \
        err_push(var, __FILE__, __LINE__); \
        goto handler                       \
    } while (0)
```

Then you can implement your error handling like this

```c
err_t *fun_a(void)
{
    bool result;
    err_t *err;

    result = fail();
    if (!result) ERR_GOTO_NEW(on_error, err, ERR_FAIL);

    err = NULL;

on_error:
    return err;
}

err_t *fun_b(void)
{
    err_t *err;

    err = fun_a();
    if (!err_is_ok(err)) ERR_GOTO(on_error, err);

    err = NULL;

on_error:
    return err;
}
```

It requires a little bit of setup in you code base, but the upside is that it
allows you to trace back your error through the call stack. I've added a more
complete example on [GitHub][github].

The exact approach for error handling need to be evaluated for each project,
there is not a one size fit all solution here. But it's worth knowing what kind
of approaches that are available. 

[github]:https://github.com/ElektroKotte/err-test
<!-- vim: set tw=80 et ts=4 ss=4 sw=4 : -->
