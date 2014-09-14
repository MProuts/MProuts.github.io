---
layout: post
title: "Find And Replace From Outside Of Files"
date: 2014-09-13 22:10:03 -0400
comments: true
categories: perl, bash, ack
---

Toss this puppy in your ```.bashrc``` for string substitution from
outside of files.


```sh
function ack_sub {
  ack -l $1 $3 | xargs perl -pi -E 's/'$1'/'$2'/g'
}
```

### Syntax

> ``` ack_sub <target> <substitution> <directory>```
