---
layout: post
title: "Rails: How To Run Tests Based On A Search Pattern"
date: 2016-03-29 13:39:11 -0400
comments: true
categories: Ruby, Rails, Ack, Testing
---

This came up while I was doing a refactoring that reached across
multiple files.  I wanted a way to make some change, then run all of the
tests that contain any reference to a particular method.

### TL;DR

Replace `<pattern>` with your search term.

```sh
ack <pattern> test/ -l | xargs -I{} ruby -I"lib:test" {}
```

### Explanation

To break this apart a little bit, I knew that you can get `ack` to give you
a list of file names that match a pattern by passing it the `-l` option.
So that first clause is pretty straightforward -- look for `<pattern>`
inside files in the `test/` directory, and give me back a list of file
names.

```sh
ack <pattern> test/ -l
```

I'm used to using this command to run a single test:

```sh
TEST=<filepath> rake test
```

However, in order to get this working with `xargs`, I had to switch to the slighlty
uglier version (which, if you look, is what rake is doing internally
anyway):

```sh
ruby -I"lib:test" <filepath>
```

`xarg`'s `-I` argument basically just lets you specify where an
argument fits inside shell command that's receiving
it.  In this case, `{}` in `ruby -I"lib:test" {}` gets replaced once in
succession by each of the file names we found with in the first clause
with `ack`.
