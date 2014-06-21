---
layout: post
title: "check for stray pry calls before commit"
date: 2014-06-20 22:59:36 -0400
comments: true
categories: git
---

Tired of accidentally checking ```binding.pry``` calls in to source?  Here's a
script that will take care of checking for them on your behalf.  Place
this in your project's ```.git/hooks``` directory in a file called ```pre-commit```.



### .git/hooks/pre-commit
```sh
#!/bin/sh

git diff --cached --name-only | \
    GREP_COLOR='4;5;37;41' xargs grep --color --with-filename -n pry && \
    echo 'COMMIT REJECTED' && \
    exit 1
exit 0
```
*Note*: Don't forget to ```chmod pre-commit u+x``` to make this file
executable.

Let's step through this line by line. 

```sh
#!/bin/sh
```

First, we use a shebang directive
to specify the interpreter that will be used to run the code in this file.  Since
we're writing a shell script we specify ```/bin/sh```.

```sh
git diff --cached --name-only | \
```

 ```git diff``` shows us any differences between the code in working directory
and our previous commit.  We pass it the ```--cached``` flag to show
changes to files that have already been staged.  The ```--name-only``` flag will
print just the names of files that have been changed, rather than the
actual changed contents of those files.

```sh
GREP_COLOR='4;5;37;41' xargs grep --color --with-filename -n pry && \
```

 ```GREP_COLOR``` is an environment variable that determines the colors
 ```grep``` uses to highlight matches when the ```--color``` flag is given.  We
use ```xargs``` to feed the individual file names that resulted from the previous line to ```grep pry```.  The ```--with-filename``` and ```-n``` flags add the
file name and line number to the results, respectively.   We use the
logical operator ```&&``` to execute the next line of code only if the
current line runs successfully.  (I.e. If ```grep``` doesn't find any
occurances of 'pry', the chain stops here.)

```sh
echo 'COMMIT REJECTED' && \
exit 1
```
If grep does find a 'pry' anywhere in our staged changes, we print an error message and exit with a status
of ```1``` for 'error', which tells ```git``` to terminate the commit.

```sh
exit 0
```
Otherwise, we exit with a status of ```0``` for 'successful', which tells ```git``` to go ahead with the commit.

### Make This Hook Global

To make this hook global, run the following command and copy the script
into ```~/.git_template/hooks```.  Thereafter, every time you call git init, it will be included in your project's ```.git/hooks``` directory.

    $ git config --global init.templatedir '~/.git_template'

    $ cp /myproject/.git/hooks/pre-commit ~/.git_template/hooks/pre-commit

And there you have it.
