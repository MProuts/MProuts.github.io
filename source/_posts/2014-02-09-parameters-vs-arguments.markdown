---
layout: post
title: "Parameters vs. Arguments"
date: 2014-02-09 13:20:04 -0500
comments: true
categories: 
---

> "The beginning of wisdom is to call things by their proper names."
> 
> -Confucius  

Why are two concepts so basic to everything we do with computers so hard for most of us to differentiate?  The short answer is that it's often not a useful distinction, so we abstract it out of our thought processes in order to make room for other things.  The long answer follows.  (Read on, interested reader!)  

In order to understand why the distinciton feels unnatural, we first have to understand the distinction itself.  Here goes: 

###Parameters
Parameters exist in *method declarations*.  They are placeholders for what later gets passed into the method.  Syntactically, they say "something goes here."  

```ruby
def plate_food(plate_goes_here,  food_goes_here)
	# code to place food on plate
end
```

In this example, ```plate_goes_here``` and ```food_goes_here``` are parameters.



###Arguments

Arguments, on the other hand, exist in method *calls*.   They **are** the actual things that get passed to a method.  

```ruby
my_plate = plate.new "porcelain"
some_food = food.new "kimchi"

plate_food(my_plate, some_food)
```

In this example, ```my_plate``` and ```some_food``` are arguments.

###How They're Related

In programming, methods are for storing behavior.  We "parameterize" methods in order to allow them to interact with data without having to specify that data.  That way, we can use the method over and over again, passing in different pieces of data at our whim.  *Parameters* act as placeholders in the method definition for hard data that is passed as *arguments* every time the method is actually invoked.  

> parameter:hypothetical::argument:concrete  

###The Confusing Part...

This distinction becomes murky as soon as we begin our programming careers, because we tend to name our placeholders (logically) for the things that will eventually take their place.  

To return to the previous example, instead of naming our parameters ```plate_goes_here``` and ```food_goes_here```, we go with ```plate``` and ```food``` instead.  

```ruby
def plate_food(plate,  food)
	# code to place food on plate
end
```

And just like that, the seam we intentionally dug out earlier has been rehidden.  And hidden it remains for 99.999% of our programming careers.  

A particularly explicit example of this placetaker-for-placeholder conflation is literally a convention in Ruby:


```ruby
def clean_plates(*args)
	args.each { |plate| plate.wash }
end
```

Here, we literally name our parameter "args."   *Could* these concepts be less practically distinct, I ask you?  

###...And Is Understanding This *All* That Important?

Practically, probably not.

If you have a bent for philosophy or linguistics, this is the type of conceptual relationship you might explore further in an introductory course in [semiotics](http://en.wikipedia.org/wiki/Semiotics) or [semantics](http://en.wikipedia.org/wiki/Semantics).  

But my sense is that this distinction is muddled in most higher-level applied programming *by design*.  After all, given the option to hide a conceptual layer from ourselves, most programmers take it without even thinking about it.  





