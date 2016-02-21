---
layout: post
title: "SSH Tunneling"
date: 2015-06-07 10:36:28 -0500
comments: true
categories: unix, ssh
---

Say you have two remote servers, **A** and **B**. Let’s say your machine can
access **A** but not **B** (via SSH), and from **A** you can access **B**. To start a
terminal session on **B** from your machine, you could SSH onto **A** first,
then SSH onto **B** from there.

```
[me@localhost ~]# ssh root@host_a

Welcome to A!
Last login: Sun Jun  7 17:45:36 2015
[root@host_a ~]# ssh ubuntu@host_b

Welcome to B!
Last login: Sun Jun  7 17:45:36 2015
[ubuntu@host_b ~]# ...
```

This two-step process works fine for anything you might want to do
manually on **B**, but what if you need to connect directly from you machine
to **B** without the intermediate step of first connecting to **A**?

For example, maybe you want to run a local rails development server but
connect it to a database at **B**’s port `3306`. You can accomplish this via
ssh tunneling.

SSH tunneling allows you to forward two-way traffic *from* a port on your
local machine *to* a port on a remote machine *through* an ssh connection to
an intermediate machine (gaining the access rights of the intermediate
machine along the way).

You can set up an ssh tunnel by passing the `-L` option to the `ssh` command
line utility in the following form:

`ssh -L <local_port>:<end_host>:<end_port> <intermediate_user>@<intermediate_host>`

Running this command opens a terminal session on the intermediate host.
The port forwarding will continue as long as this session remains open.

Returning to our example:

```
[me@localhost ~]# ssh -L 3307:host_b:3306 root@host_a

Welcome to A!
Last login: Sun Jun  7 17:45:36 2015
[root@host_a ~]# ...
```

#### config/database.yml

```
development:
  ...
  username: deploy
  password: 1234
  host: 127.0.0.1
  port: 3307

...
```

If we then fire up our rails server and point our browser at
`127.0.0.1:3000`, we’ll see our app connected to the database at
`host_b:3306`.

