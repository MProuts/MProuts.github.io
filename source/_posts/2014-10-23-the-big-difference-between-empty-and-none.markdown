---
layout: post
title: "the big difference between empty and none"
date: 2014-10-23 18:39:29 -0400
comments: true
categories: ActiveRecord, Ruby, Enumerable, ActiveRecord::Relation
---

I recently made this mistake:

```
if User.first.records.none?
...
```

What's wrong this this code?  
