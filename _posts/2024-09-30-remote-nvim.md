---
layout: post
title:  "Neovim over ssh tunnel"
date:   2024-09-30
author: Emil Ohlsson
categories: neovim ssh
---
Every now and then I find myself wanting to work with neovim remotely. Sure, I
can `ssh`, and then start `nvim` and have it work that way. But there is some
form of convenience of being able to just use the builtin server functionality.

On client
```sh
ssh -L localhost:4567:localhost:6666 server-addr \
  "SOME_VAR=value nvim --headless --listen localhost:6666"
```

This will launch neovim on the server, and have it listen locally on port
`6666`,
while `ssh` opens port `4567` which will forward traffic to port `6666` on the
server (neovim that is).

On the client you can now do a remote connection to neovim using
```sh
nvim --remote-ui --server localhost:4567
```

It's worth pointing out that you can have ssh simply just forwarding a port,
wihtout actually launching any program on the server. To do this, pass the
[`-N`][ssh-N] flag (Do not open shell/execute remote command), and optionally
[`-f`][ssh-f] (run in background and redirect output to `/dev/null`).
```sh
ssh -fNL localhost:<local_port>:localhost:<server_port> <server-addr>
```
The downside to this is that you don't know the process id of `ssh`, so you can
have a hard time terminating it (need to use `ps` or similar). Instead you can
set up control sockets

## A more advanced setup using control sockets
To be able to control a connection that is running in the background you can
pass runtime flags to set things up, but this is pretty tedious. So to make this
a bit more convenient you can store a host specfic configuration for `ssh`. To
do this create a directory for control paths, and a config file
```sh
mkdir -pm 700 ~/.ssh/controlmasters
```

Next, edit the file `~/.ssh/config` to contain the following
```
Host <nickname>
        HostName <actual server address>
        ControlPath ~/.ssh/controlmasters/%r@%h:%p
        ControlMaster auto
        ControlPersist 10m
```

If you created `~/.ssh/config`, make sure that you set the permissions to
`0600`. `ssh` is pretty picky about this.

Now you can re-establish the ssh tunnel using
```sh
ssh -fNL localhost:4567:localhost:6666 <nickname> \
  "SOME_VAR=value nvim --headless --listen localhost:6666"
```

When you run the above command, you should get a file called
`~/.ssh/controlmasters/<user>@<hostname>:<port>`

This allows you to check status and terminate connections using commands like
```sh
ssh -O check <nickname>
ssh -O exit <nickname>
```

`check` will provide you with process id and status, `exit` will ask the control
master to terminate. You can read the [documentation][ssh-O] about how more
details.

This process set up here is called _multiplexing_, and it's described better in
the [OpenSSH cookbook][cookbook_multiplexing]

Finally, it's worth pointing out that this feature is only triggered if your
using the configured nickname.

Another benefit of this configuration is that if you're doing multiple
connections to the same target, then the subsequent connections are much faster
to set up, as you simply re-use the existing connection.

[cookbook_multiplexing]: https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing#Setting_Up_Multiplexing
[ssh-f]: https://man.openbsd.org/ssh.1#f
[ssh-O]: https://man.openbsd.org/ssh.1#O
[ssh-L]: https://man.openbsd.org/ssh.1#L
[ssh-N]: https://man.openbsd.org/ssh.1#N

<!-- vim: set et ts=2 sw=2 ss=2 tw=80 : -->
