---
layout: post
title: "Manage Your Dotfiles With Git"
date: 2014-02-04 22:55:48 -0500
comments: true
categories: [dotfiles, shell scripting, git]
---

After spending an unreasonable amount of time tinkering with ```LSCOLORS``` and ```PS1``` in my ```.bash_profile``` (think blinking pink unicode coffee cup), I made a typo and broke the file.  The error was deep inside my long, opaque ```PS1``` string, and I eventually decided to scrap the broken version and start over.  I also decided to put my dotfiles under version control.

This turned out to be a little trickier than I expected...

Separate Your Dots
------------------

The first problem I encountered is that your system expects dotfiles to live in your home directory at paths like ```~/.vimrc```. This isn't a great place for a git repository because, well, that's where *ALL* of the rest of your user files live.  On the other hand, if you move them into their own directory, 

    $ mkdir ~/dotfiles/
    $ mv ~/.bash_profile /dotfiles/bash_profile

they will no longer be accessible to programs that use them to load your customization.

Symlinks
--------

Enter symlinks.  Symlinks basically let you create a named pointer (or *symbolic link*) to a file at a different location in the file system.  When the system tries to open this pointer, it will link to the target file, and open that instead.  In our case, this means that we can stash all of our resource files in ```~/dotfiles``` and create links to make them accessible to programs looking for them in ```~/```. Here's the line of code that makes that happen.

    $ ln -s /dotfiles/bash_profile ~/.bash_profile

This is basically just saying: 
"Create a link from stashed file ```/dotfiles/bash_profile``` to its original location at ```~/.bash_profile```.  (I decided to remove the dot from the stashed file for convenience.)

Git It!
-------

With the symlink in place, our ```bash_profile``` should now be working from its new location.  (Note: You may need to close and reopen terminal to see a change). All that's left to do is initialize our git repo and make our first commit.


    $ cd ~/dotfiles
    $ git init
    $ git add .
    $ git commit -m "Initial commit"


Syntax Highlighting
-------------------

You'll notice that as a side effect of moving our ```bash_profile```, we no longer get pretty syntax highlighting when we open it in an editor. 

  **dotfiles/bash_profile**

    PATH="/Applications/Postgres.app/Contents/MacOS/bin:$PATH"

    CDPATH=~/dev/

    [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

    export PS1="\W ☕ "
    export GREP_OPTIONS='--color=always'
    export LSCOLORS="GxGxBxDxCxEgEdxbxgxcxd"

The solution is to explicitly tell the editor the file type (a shell script) by appending ```.sh``` to the filename.

  **dotfiles/bash_profile.sh**

```bash


PATH="/Applications/Postgres.app/Contents/MacOS/bin:$PATH"

CDPATH=~/dev/

[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*

export PS1="\W ☕ "
export GREP_OPTIONS='--color=always'
export LSCOLORS="GxGxBxDxCxEgEdxbxgxcxd"
```

Aaaaah. Much better. 

Generate Symlinks with a Script
-----------------------------

Great, so now version control and syntax hightlighting are working, but we've introduced another problem.  Namely, it's a pain to manually maintain these symlinks.  As your list of dotfiles grows, or when you change computers, it would be nice to have a way to dynamically generate links to these stashed resource files. Solution: write a little shell script to link all of the files in ```/dotfiles``` to the home directory for you.  I just threw this in the same directory.

**linkdotfiles.sh**
```
dir=~/dotfiles          # dotfiles directory

for file in $dir/*; do
  if [ $(basename $file .sh) != "linkdotfiles" ]
  then
    echo "Linking" $(basename $file) "..."
    ln -s $file ~/.$(basename $file .sh)
  fi
done
echo "... all done!"
```

To run: 
    $ sh linkdotfiles.sh

All done.  :)