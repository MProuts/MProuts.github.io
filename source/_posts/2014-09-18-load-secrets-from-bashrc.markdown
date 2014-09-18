---
layout: post
title: "Load .secrets from .bashrc"
date: 2014-09-18 18:44:28 -0400
comments: true
categories: bash, .bashrc, shell
---

Your shell resource file is a great place to set local environment
variables for use in development (things like API keys, etc.) -- that is,
until you decide to check it into source control.

**Solution**:
Toss those variable assignments into a file called ```.secrets``` and use the shell
command ```source``` to pull them into your environment when the resource
file gets loaded.

### .secrets

```sh
export AWS_ACCESS_KEY_ID=imnottellingimnottelling
export AWS_SECRET_ACCESS_KEY=myfavoritefoodissushi
```

### .bashrc

```sh
source ~/.secrets
```

And now you can safely push your ```.bashrc``` up to Github.

...after you create a little repo for your [dotfiles](/blog/2014/02/04/manage-your-dotfiles-with-git/), that is! ;)
