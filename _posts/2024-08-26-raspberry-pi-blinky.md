---
layout: post
title:  "Blinky with raspberry pi pico on fedora"
date:   2024-08-26
author: Emil Ohlsson
categories: c embedded
---

So I've just gotten one of the new raspberry pi picos version 2, and since it's
been a long time since I've been working with proper emdedded devices I wanted
to take stab at getting a blinky program up and running. This is a good way to
get started and familiarizing myself with the development environment.

So since I'm using Fedora linux I need install some other packages than what's
listed on the SDK description. I already have a bunch of "development stuff"
installed, such as `git`, `cmake` etc.

I wanted to be able to write in C or C++, so I figured that the `pico-sdk` would
be a good starting point for me. This requires arm compilers to be available.
These can be installed with
```sh
sudo dnf install gcc-arm-linux-gnu \
                 arm-none-eabi-gcc-cs-c++ \
                 arm-none-eabi-gcc-cs \
                 arm-none-eabi-binutils \
                 arm-none-eabi-newlib 
```

Afterwards you need to clone the SDK and you can also clone the examples. This
would be
```sh
git clone --recursive https://github.com/raspberrypi/pico-sdk.git
git clone --recursive https://github.com/raspberrypi/pico-examples.git
```

Just as an example you can copy the blinky code example to a new directory
```sh
cp -r pico-examples/blink my-blinky
cd my-blinky
```

The pico SDK readme lists a couple a ways to use the SDK, and I've kind of done
a mixture of a couple of ways. Edit the `CMakeLists.txt` using the steps
descibred by the pico SDK readme. Prepend the `CMakeLists.txt` with the following
lines
```cmake
cmake_minimum_required(VERSION 3.13)

include(/home/kotte/workspace/pico-sdk/pico_sdk_init.cmake)

project(blink)

pico_sdk_init()
```

Finally, remove the `example_auto_set_url(blink)` line at the end of the file,
as this seem to be  tied to the examples build system, and at the moment I don't
care about that.

Finally, given that this board is a raspberry pi pico 2, build with the
following commands

```sh
cmake -B build -DPICO_BOARD=pico2
cmake --build build
```
This should have produced `build/blink.uf2`, which is the binary file expected
by the Raspberry pi pico bootloader.

To program the raspberry pi, hold the `BOOTSEL` button, and connect it to the
computer.

Mount the image using
```sh
udisksctl mount -b /dev/disk/by-label/RP2350
# This will print the mount point, which for me is /run/media/$USER/RP2350
cp build/blink.uf2 /run/media/$USER/RP2350
```

Now, if everything is done correctly, the board should now be blinking!

[automatic variables]: https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
[implicit rules]: https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
