---
layout: post
title: "Star Register Not Working In Vim?"
date: 2014-05-01 11:19:04 -0600
comments: true
categories: vim, homebrew, vim registers, star register
---

The star register provides a convenient way to read and write from the system clipboard in vim.  (For example: ``` "*yy``` in vim, ```ctrl-v``` in another application.)

For this to work, you need a version of vim
compiled with the ```clipboard``` feature included.  You can easily check
the features included in your vim version by running:

```
$ vim --version
...
-clipboard       +iconv           +path_extra      -toolbar
...
```

If you see ``` +clipboard```, you're in good shape.

Elsif you see ```-clipboard```, read on for the fix.

1) Get the latest version of vim.
```
brew install vim
```

2) Check which local version of vim is being used.
```
which vim
/usr/bin/vim
```

3) If you don't see ```/usr/local/bin/vim```,
add the following lines to ```~/.bashrc``` or ```~/.zshrc```:

```
# Prefer Homebrew binaries
export PATH=/usr/local/bin:$PATH
```

4) Run ```vim --version``` again to make sure it worked.  
```
$ vim --version 
...
+clipboard       +iconv           +path_extra      -toolbar
...
```

Success!

