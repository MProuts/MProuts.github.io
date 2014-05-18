---
layout: post
title: "hello, bower"
date: 2014-05-18 11:05:16 -0400
comments: true
categories: 
---

Bower is a simple package manager for front-end assets like CSS and
Javascript. It provides an elegant solution for including third-party assets and libraries (e.g.
Bootstrap, jQuery) into different projects on your machine.  Here's how it
works:

###Get Bower
```
$ npm install -g bower
```

###Install a Package
Bower installs packages on a project-by-project basis.  Make sure
you're inside of the project directory where you want a package installed first,
then run:

```
$ bower install <package>
```

This command creates a directory inside your project called ```bower-components/```
containing the package you've installed, as well as any dependencies.

###Use a Package
To use a Bower package, simply reference the appropriate files in ```bower-components/``` manually.  

```
<script src="/bower_components/jquery/jquery.js"></script>
```

###List Packages

```
$ bower list
```

Use this command to list all packages installed on a given project.  Again, the list of
installed packages will vary on a project-to-project basis.
