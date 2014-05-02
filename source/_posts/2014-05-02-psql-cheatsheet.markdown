---
layout: post
title: "psql cheatsheet"
date: 2014-05-02 13:48:12 -0600
comments: true
categories: postgresql
---
Django development requires that you know your way around your database
console.  Here are the basics for PostgreSQL:

###start
```
psql            #Start interactive terminal
```

###database
```
\list 		            #List databases
\connect <db>            #Switch connection

SELECT 
current_database();	   #Show database connection
```

###tables
```
\d			   #List tables
\d <table>      #List table columns
```

###help
```
\?              #List backslash commands
\help           #List SQL commands
```

###quit
```
\q			   #Quit  
```
