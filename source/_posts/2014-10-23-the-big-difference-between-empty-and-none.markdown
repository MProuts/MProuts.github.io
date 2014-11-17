---
layout: post
title: "the big difference between empty and none"
date: 2014-10-23 18:39:29 -0400
comments: true
categories: ActiveRecord, Ruby, Enumerable, ActiveRecord::Relation
---

Given an ActiveRecord class with a has_many association...

```ruby
class User < ActiveRecord::Base
  has_many :records
  ...
end
```

...this is a mistake.

```ruby
user = User.first
user.records.none?
```

Why, you ask?

Let's step back for a second.  On a less-abstract level, the correct way to answer the
question, "Are there any records that belong to this user?" is to
fire a SQL statement with a ```SELECT count(*)``` clause, and see whether the result is
greater than 0.  That's what we want to have happen behind the scenes.

However, that's not what's happening when we call ```none?```, because
```none?``` is not defined on ```ActiveRecord::Relation``` (the
fancy/magical Rails class that is responsible for abstracting away cross-table SQL queries), but rather, it is
defined on the plain old Ruby module ```Enumerable```.

So, what you might assume (as I did) is doing something smart and
reasonable (i.e.
performing a SQL count, and comparing the result with zero), is actually
doing something **VERY** stupid (i.e. loading a ton of Ruby objects into memory
and performing a count on them there).  You can imagine that if users had thousands of
records each, this line could constitutes a serious resource drain for your application.

###Solution

> Know thy ```ActiveRecord::Relation``` methods.

The method you were probably reaching for was ```ActiveRecord::Relation#empty?```, which
performs a SQL count().  For reference, here's a breakdown of some other
count-related methods on ```Enumerable``` and their ```ActiveRecord::Relation``` counterparts.

| Question | Enumberable Method | ActiveRecord::Relation Method
| -------- | ------------------ | -----------------------------
| **More than zero** objects in collection? | ```any?``` | ```any?```
| **Zero** objects in collection? | ```none?``` | ```empty?```
| **More than one** object in collection? | ```x``` | ```many?```
| **Exactly one** object the collection?   | ```one?``` | ```x```

<br>
