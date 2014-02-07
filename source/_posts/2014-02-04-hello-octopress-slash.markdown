---
layout: post
title: "Manage Your Dotfiles With Git"
date: 2014-02-04 22:55:48 -0500
comments: true
categories: [dotfiles, shell scripting, git]
---

This is a Header
================

After spending an unreasonable amount of time tinkering with LSCOLORS and PS1 in my .bash_profile (think blinking unicode coffee cup), I made a typo and broke the file.  The error was deep inside the long, opaque PS1 string, and I eventually decided to scrap the broken version and start over.  I also decided to put it my dotfiles under version control.

This turned out to be trickier than I expected. 

The first problem I encountered is that your system expects dotfiles to live in your home directory at paths like ~/.vimrc. This isn't a great place for a git repository because well, that's where ALL of the rest of your user files live.  On the other hand, if you move them into their own directory, 

mkdir ~/dotfiles/
mv ~/.bash_profile /dotfile/bash_profile 

the programs that use these files to load your customization will still look for them in the home directory, and no longer find them there.

Enter symlinks.  Symlinks basically let you create a named pointer to file at a different location in the file system.  When the system tries to open this pointer, it will link to the target file.  In our case, this means that we can stash all of our dotfiles our dotfile/ directory and create links to make them accessible in ~/.  Here's the line of code that makes that happen.

ln -s /dotfile/bash_profile ~/.bash_profile

This is basically just saying: "Create a link from my stashed file at /dotfiles/bash_profile to its original location at ~/.bash_profile.  (I decided to remove the dot from the stashed file for convenience.)

With this knowledge in hand, all that's left to do is initialize our git repo and make our first commit.

cd ~/dotfiles
git init
git add .
git commit -m "Initial commit"

Sidenote: you'll notice after you move a dotfile that syntax highlighting no longer happens automatically. 

<example>

The solution is to explicitly tell the editor the file type (a shell script) by appending .sh to the filename.

<example>

Great, so now version control and syntax hightlighting are working, but we've introduced another problem.  Namely, it's a pain to manually maintain these symlinks.  As your list of dotfiles grows, or when you change computers, it would be nice to have a way to dynamically generate links to these stashed resource files. Solution: write a little shell script to link all of the files in dotfiles/ for you.

dotfilesymlinks.sh

dir=~/dotfiles          # dotfiles directory

for file in $dir/*; do
  if [ $(basename $file .sh) != "linkdotfiles" ]
  then
    echo "Linking" $(basename $file) "..."
    ln -s $file ~/.$(basename $file .sh)
  fi
done
echo "... all done!"

All done.  :)