---
layout: post
title: "Find and Replace from Shell with Sed"
date: 2014-11-18 11:26:18 -0500
comments: true
categories: sed find-and-replace shell command-line
---

This came up again while I was in the process of upgrading our app from Bootstrap 2.3 to 3.0.  Much of the change involved updating CSS classes.  Bootstrap 3's new grid system necessitated a slew of changes in the form ```span6``` -> ```col-md-6```, where a prefix before a number changes, but the number
stays the same.

In such cases, you need to include the number character in the regex
you're matching against (e.g. so that you don't inadvertently change a bunch
of ```<span>``` tags into ```<col-md->``` tags). At the same time, you
need include the number portion in the replacement string -- i.e.
 ```class=span1``` becomes ```class=col-md-1```, ```class=span12```
becomes ```class=col-md-12```, etc.

Both of these requirements call for using regex captures.
However, there were some problems getting regex captures to work with the [ack_sub](/blog/2014/09/13/find-and-replace-from-outside-of-files/) function I created earlier (namely, both perl regex captures and bash arguments assign to
variables prefixed with dollar signs like ```$1```.)

After a little research, a popular unix utility called ```sed```
surfaced as a tidy solution.  Using the example I gave earlier, the command to change all ```span6``` classes to ```col-md-6``` ones is
as follows.

```sh
find . -type f | xargs sed -i '' -E 's/span([0-9]+)/col-md-\1/g';
```

Let's break this down.  To start, ```sed``` accepts a vi-style regex substitution
command as its first argument.

```sh
echo "hello" | sed 's/ello/i/'
hi
```

It also accepts file name as a second argument.

```sh
echo "hello" > test.txt
sed 's/ello/i/' test.txt
hi
```

**Note**: In this form, ```sed``` reads from the file, but does not edit it.

```sh
cat test.txt
hello
```

In order to actually *edit* the file, we'll pass ```sed``` the ```-i``` flag (for in-place file editing).
With the ```-i``` flag, ```sed``` takes three arguments -- the extension to
append to a backup copy of the original file, followed by the
substitution command string, followed by the name of the file to execute the command on. If you want to edit the file without creating a backup, you can
pass an empty string as the first argument.

**Warning**: This is risky.  Only try this at
home if the files you're editing are backed up and/or under version
control.

```sh
sed -i '' 's/ello/i/' test.txt

cat test.txt
hi
```
Great, now in order to use regex captures as we discussed earlier,
we'll pass ```sed``` the ```-E``` flag, which enables advanced regex features.  When used in combination with the ```-i``` flag, the ```-E``` flag goes between the first and second arguments.

**Note**: ```sed``` regexes don't support the digit-matching escape character ```\d```.  We'll use the character range ```[0-9]``` instead.

```
echo "span6" > test.txt
echo "span12" >> test.txt
cat test.txt
span6
span12

sed -i '' -E 's/span([0-9])/col-md-\1/g' test.txt
cat test.txt
col-md-6
col-md-12
```

Beautiful.  Now we can run the command recursively on all of the files
beneath a given directory using the ```find``` in combination with
 ```xargs```.

```
find app/views/ -type f | xargs sed -i '' -E 's/span([0-9])/col-md-\1/g'
```

Cool.  But be really careful.  In-place file editing is scary.
