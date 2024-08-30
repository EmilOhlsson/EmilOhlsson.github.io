---
layout: post
title:  "Setting up raspberry pi pico development"
date:   2024-08-29
author: Emil Ohlsson
categories: c embedded
---

Last time I wrote about how to build a firmware image for the pico 2. This time
the plan is to set up debugging using openocd, and hook it into my development
environment

This process uses the raspberry pi debug probe. The rest of the text assumes
that it's connected to both your computer and you debug target, and permissions
are set up properly. This should mostly be a matter of creating the udev file
`/etc/udev/rules.d/20-debug-probe.rules` with something like the following
content:
```
ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="000c", MODE="660", GROUP="plugdev", TAG+="uaccess"
```

Ideally we don't want to have to be root to be able use the debug probe, and the
above configuration should make sure of that.

Now, after some experimentation I found a couple of gotchas, that might make the
setup process different in the future. For the time being, the latest stable
release of OpenOCD, 0.12.0, does not properly support the pico. The problem is
that stopping the execution causes sleep functions to freeze. This meant that I
instead had to build OpenOCD from source, which causes some other headache. Long
story short: libgpiod 2.x is not backward compatible with previous versions, and
OpenOCD uses 1.x. 

There is a [github issue] describing what to look for, and what the underlying
problem is with using latest stable version of OpenOCD

So normally, you would start with installing OpenOCD
```sh
sudo dnf install openocd
```

Now instead we might as well clone the [openocd fork] made by the Raspberry pi
foundation. When we build OpenOCD we need to disable some warnings, since the
recent version of gcc on Fedora causes some build errors due to newer warnings.
When building openocd you also need to make sure that some dependencies are
available.

```sh
sudo dnf install libtool libusb1-devel hidapi-devel
```

Then clone, configure, compile and install. The steps below will install in
`$HOME/local`
```sh
git clone --recursive openocd openocd-rbpi
cd openocd-rbpi
./bootstrap
CFLAGS=-Wnoerror ./configure --prefix=$HOME/local \
                             --program-suffix=-pico \
                             --enable-cmsis-dap \
                             --disable-linuxgpiod
make -j$(nproc)
make install
```

If you install OpenOCD using the options above you will have a local install
called `openocd-pico`.

For this example we will build the blink example for the original pico. So using
the steps from the previous post, build using the following commands:
```sh
cmake -DCMAKE_BUILD_TYPE=Debug -B build -DPICO_BOARD=pico
cmake --build build
```

The Debug part is important, as that allows `gdb` to read debug symbols.

Now you can flash the device using openocd using the following commands:
```sh
export OOCDS=$HOME/local/share/openocd/scripts
openocd-pico \
  -f $OODCS/interface/cmsis-dap.cfg \
  -f $OODCS/target/rp2040.cfg       \
  -c "adapter speed 5000"    \
  -c "program build/blink.elf verify reset exit"
```

If this worked, and your device started blinking the onboard LED, you should
have everything working, and now it's "only software".

For getting things working with `gdb` you want to launch OpenOCD like this:
```sh
export OOCDS=$HOME/local/share/openocd/scripts
openocd-pico \
  -f $OODCS/interface/cmsis-dap.cfg \
  -f $OODCS/target/rp2040.cfg       \
  -c "adapter speed 5000"
```

You should have log output stating that OpenOCD is listening on port 3333.
Launch gdb, load debug symbols, and connect to the debug server
```sh
gdb build/blink.elf
```

In gdb, connecto the server
```gdb
target extended-remote :3333
break main
run
```

You should now be able to run gdb as normal. And you can again do the same flash
operations by doing monitor commands

```gdb
monitor program build/blink.elf verify reset exit
```

[openocd fork]: https://github.com/raspberrypi/openocd
[github issue]: https://github.com/raspberrypi/pico-sdk/issues/1622

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
