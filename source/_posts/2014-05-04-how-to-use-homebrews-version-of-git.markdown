---
layout: post
title: "How To Use Homebrew's Version of Git"
date: 2014-05-04 13:20:02 -0600
comments: true
categories: 
---

If you’re using Homebrew with OSX, chances are you have multiple copies of the same software packages living on your machine. With this in mind, it's important to know which version of a particular package gets run when you type a command at the command line.  For example, when you type ```git status```, are you running OSX's version of git or Homebrew's version?

###OSX Packages

OSX’s binaries live in ```/usr/bin```.  They are the defaults that either came
preinstalled in your machine, or were bundled with Xcode.
```
$ ls /usr/bin
```

###Homebrew Packages

Under the covers, Homebrew installs packages or ‘kegs’ in their own directories in ```/usr/local/Cellar```.
```
$ ls /usr/local/Cellar
```

For convenience, Homebrew symlinks binaries nested within these directories to ```/usr/local/bin```.  These links are how you access Homebrew binaries.

```
$ ls /usr/local/bin
```

###Which Package?
Which of these two versions gets run is determined by your PATH variable.
```
$ which git
/usr/bin/git
```
If you take a look at your global PATH configuration by running ```cat etc/paths``` you'll see that binaries in ```/usr/bin``` are
run before binaries in ```/usr/local/bin``` by default.

```
$ cat etc/paths
/usr/bin
/bin
/usr/sbin
/sbin
/usr/local/bin
```

You can move Homebrew's binaries to the front of the line by adding this
to your preferred shell's resource file (```bashrc``` or ```zshrc```).

```
export PATH=/usr/local/bin:$PATH
```

Voila! 
```
$ which git 
/usr/local/bin/git
```
