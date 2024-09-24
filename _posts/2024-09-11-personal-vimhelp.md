---
layout: post
title:  "Personal vimhelp"
date:   2024-09-11
author: Emil Ohlsson
categories: documentation
---
I've been using vim and neovim for several years, and over the time I've
accumulated several useful plugins. I do try to keep the amount of plugins to a
minimum, as there are several reasons not to use plugins (security, setup,
mental and computing overhead). Still, there are several really good plugins
there, and they are part of what makes Vim and Neovim great.

So there are several times where I need to consult my configuration files as to
remember how certain plugins are used. Every now and then I might also forget
how I've structured certain setup. So I figured that why not simply create my
own help files, available from within vim.

The documentation of help files can be found in `:help help`, 

So to accomplish this I created a `doc` folder within my vim configuration
directory, and created a file `my-usage.txt` within it. The syntax for the help
files can pretty easily be understood by simply reading the files. There is
documentation within `:help help-writing` on how to write help files, and here
is a simple example

```
*filename.txt* What does this file describe

Some more text

==============================================================================
Some section title			                               			*my-section*

Even more text, |link|, `command`

------------------------------------------------------------------------------
Some sub-section			                                 			*my-commands*

more text
```

Syntax is best described in `:help notation`.

Once you've created your personal documentation file you need to run `:helptags
ALL` as to generate tag files used by vim to find the sections. So once you've
created tag files on the text above you can find `my-..` using autocomplete with
`:help my-`. Pretty handy!

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
