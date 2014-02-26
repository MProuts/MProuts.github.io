---
layout: post
title: "7 Colloquialisms From Code"
date: 2014-02-24 15:19:09 -0500
comments: true
categories: 
---

Beyond specific technical jargon ("loop," "thread," "iteration"), the language of programming languages is rich in strange historical colloquialisms.  Here are 8 you might not know...

Spaghetti Code: 
-----
***noun*** 1. pejoritave term for code whose flow of execution is muddled, often associated with poor usage of the GOTO keyword. *I was going to give her write access to my repo, but I hear she writes spaghetti code.*

{% img left http://upload.wikimedia.org/wikipedia/commons/9/93/Spaghetti.jpg 300 300 %}
{% img images/fortran.png 400 400%}

**See also**: 
[Ravioli Code](http://en.wikipedia.org/wiki/Spaghetti_code#Ravioli_code), 
[Lasagna Code](http://en.wikipedia.org/wiki/Spaghetti_code#Lasagna_code), 
[Spaghetti With Meatballs Code](http://en.wikipedia.org/wiki/Spaghetti_code#Spaghetti_with_meatballs),
[Macaroni Code](http://en.wikipedia.org/wiki/Spaghetti_code#Macaroni_code)


Syntactic Sugar:
--------

***noun***
1. optional syntax in a programming language that makes code more readable,
   consice or otherwise sweeter for the humans that interact with it. *I take my
   [syntactic] sugar with coffee and cream.*

{% img left http://www.usafutures.com/sugar_trading_broker.jpg 300 300 %}

```ruby

# Salty
def keep_parantheses(on); end
[].<<("Something")
4.+(4)
4.%(4)

# Sweet
def leave_parentheses off; end
[] << "Something Else"
4 + 4
4 % 4
```

Duck Typing
------

***noun***
1. A style of dynamic typing where an object's type is defined by what it can do
   rather than what it *is* (i.e. its class).  

**etymology:**
From the popular saying, "If it looks like a duck and quacks like a duck, it must
be a duck."

{% img left http://www.stanford.edu/dept/CTL/cgi-bin/academicskillscoaching/wp-content/uploads/2012/07/baby-duck.jpg 300 300 %}   

```
class Duck 
  def quack; "quack!"; end
end

class Dog
  def quack; "woofquack!"; end
end

ducks = [Duck.new, Dog.new]
ducks.each { |duck| duck.quack }
#=> ["quack!", "woofquack!"]

```
Duck Punching
-------

***noun***
1. Modifying code at runtime (e.g. overriding methods, attributes) without
   changing the underlying source code.  

**etymology:**
From the popular saying, "If it looks like a duck, but barks like a dog at runtime, you
must have punched it." ;)

{% img left http://www.justindlevine.com/wp-content/uploads/2013/11/duck-1.jpg 300 300 %}

```
class Duck 
  def quack; "quack!"; end
end

# later, at runtime...
class Duck
  def quack; "woof, ouch!"; end
end

Duck.new.quack
#=> "woof, ouch!"

```

Twiddle-Wakka
-------
**noun**
1. Mysterious squiggle-carrot operator (also known as the pessimistic operator) used to
specify versions in ruby Gemfiles.
*I see your twiddle-wakka, and raise you a scope resolution operator.*

{% img left /images/twiddle.png 300 300 %} 
```
source 'https://rubygems.org'

gem 'rails'
gem 'sqlite3'
gem 'pry'

#...

#Highest version >= 3.2.1 and < 3.3.0
gem 'sapphire', '~> 3.2.1'

#Highest version >= 2.1 and < 2.2
gem 'diamond', '~> 2.1'

```

Shaving A Yak
----------
***gerandial noun***
1. The state of being engaged in an activity that is remotely related to the original
task at hand, often after a long chain of troubleshooting digressions. *I intended to shave a yak, but pretty soon, I was filing my taxes.*

{% img left http://a-z-animals.com/media/animals/images/original/yak9.jpg 320 320 %}
```
def make_dinner
  buy_ingredients
end
def buy_ingrediens
  earn_money
end
def earn_money
  shave_yak
end
def shave_yak 
  raise "Gross, what am I even doing?"
end
```

Easter Egg
----
**noun**
1. Inside jokes and other funny superfluous tidbits intentionally left in source code by
developers for other developers to find.

{% img left /images/easter-egg.png 285 285 %}
```
class hidden_treasure
  def initialize
    ALL << self
#      .-"-.
#    .'=^=^='.
#   /=^=^=^=^=\
#  :^= HAPPY =^;      Can you spot the Easter egg
#  |^ EASTER! ^|       in this source code?
#  :^=^=^=^=^=^:  
#   \=^=^=^=^=/
#    `.=^=^=.'
#      `~~~`
  end
end
```
