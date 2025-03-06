---
layout: post
title:  "Encrypted password storage"
date:   2025-03-06
author: Emil Ohlsson
categories: misc
---

I recently came across a pretty neat tool for password storage. It is called
[password store][passwordstore]. It is a command line tool for interacting with
an encrypted password database, which uses GPG for encryption. There are plenty
of tools around for accessing passwords, though the android app was
unfortunately recently abandoned. Still, it's useful for handling keys in the
command line, without having API keys and the like pop up in the terminal
history.

There is a good outline for how to use it on the web page, but I'll outline it
here as well, as to make it easier for me to remember.

You need to start with generating your public and private keys. This can be done
with
```sh
gpg --full-gen-key
# Select encryption scheme 
# Select expiration date
# Enter your name and address
# You can leave comment empty
```

Once the above command finishes, you'll get a key identifier. You'll need this
when initializing the password storage
```sh
pass init <key id>
```

Pass also supports acting as a git repository, which is initialized as
```sh
pass git init
```

The `git` subcommand handles any `git` interaction, and this can also be used to
set upstream repositories, just like any other git repository.

Once you have keys in your database you can use
```sh
pass show VCS/github
```

to show the password, or pass `-c` to put the password in the system clipboard

## Adding keys
As they password identifiers are unencrypted, you want to make sure to not use
any private information as identifiers. As always you also want to make sure
passwords are strong. `pass` can be used to generate passwords. So we'll start
with generating an exmple password
```sh
pass generate VCS/github
```
This will then show the generated password. You can also pass the `-c` flag, to
instead send password to system clipboard, as to allow you to just paste the
generated password

You can also insert a manually entered password using
```sh
pass insert VCS/gitlab
```

You'll likely want to store user name as well, but as you don't want to leak
this information un-encrypted, it is better to enter it into the encrypted file.
This can be done using
```sh
pass edit VCS/gitlab
```

And make the content look like

```
<password>
Username: my_user@name.com
```

## Getting access on a new computer
To allow access on a new computer you first need to export your GPG keys, which
is done using
```sh
gpg --list-keys
gpg --output public.pgp --armor --export <key>
gpg --output private.pgp --armor --export-secret-key <key>
```

Transfer the keys to a new computer, and make sure the key is marked as trusted
by running
```sh
gpg --import private.pgp
gpg --import public.pgp
gpg --edit-key <email>
>> trust
>> 5
>> save
```

Then you'll need to clone the key storage using something like
```sh
git clone <url to password storage>
```

[passwordstore]: https://www.passwordstore.org/

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
