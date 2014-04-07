---
layout: post
title: "Testing with FactoryGirl and Faker"
date: 2014-03-25 19:57:39 -0400
comments: true
categories: 
---

In the context of Rails development, fixtures provide an easy way to create sample data and
make it available to your tests. Every time Rails generates a model, it
also creates a corresponding .yml in ```test/fixtures``` for your fixture code.

###test/fixtures/users.yml
```
meryl:
  name: Meryl Streep
  birthday: 1949-06-22
  occupation: Actress
```

You can access fixtures in your tests as follows:

```ruby
users(:meryl)
```

While fixtures let you weave elaborate narratives involving Meryl Streep's interactions with your site, they tend not to be very
DRY.  For example, if you want access to 12 pre-defined instances of ```User``` in your specs, you need to define 12 fixures in your
**users.yml** file.  This process can be cumbersome and error-prone.

Factory Girl
============

There's a gem for that.  

###Gemfile
```
gem 'factory_girl_rails'
```

Factory Girl is Thoughtbot's solution to this problem, and it is awesome.  Rather than hardcode your fixture data, Factory Girl lets you define factories -- methods that dynamically generate objects -- using a friendly DSL.

###spec/factories.rb
```ruby
FactoryGirl.define do
  factory :user do
    name "Meryl Streep"
    birthday "1949-06-22"
    occupation "Actress"
  end
end
```

It then exposes methods for assembling users from said factories in your tests.  

###spec/models/users.rb
```
build(:user)
#=> unpersisted version of Meryl
create(:user)
#=> persisted version of Meryl (separate object)
```

Sequences
=========

Let's say your user model validates uniqueness on its
'name' attribute, and you need to create 20 valid users in
your tests (each with a unique name).  Two 'Meryl Streep' s will no longer fly.

Luckily, Factory Girl provides a build-in fix in the form of sequences.

```
factory :user do
  sequence(:name) { |n| "Meryl Streep#{n}" }
  birthday "1949-06-22"
  occupation "Actress"
end
```

Then...

###spec/models/users.rb
```
create(:user).name
#=> "Meryl Streep1"
create(:user).name
#=> "Meryl Streep2"
create(:user).name
#=> "Meryl Streep3"

```

Strange example, but you get the picture. 

Faker
=====
Given that (possibly) not everyone loves Meryl Streep as much as I do,
wouldn't it be nice if our factory could dynamically generate unique, natural-sounding
names for each user it created?

There's a gem for that.  

###gemfile
```
gem 'faker'
```

###spec/factories
```
factory :user do
  name Faker::Name.name
  birthday "1949-06-22"
  occupation "Actress"
end
```
###spec/users_spec.rb
```
create(:user).name
#=> "Anastacio Jast"
create(:user).name
#=> "Edna Weimann"
```

Faker can fake out lots of other things as well:
```
Faker::Internet.email
#=> "rosanna_smitham@vandervort.biz"
Faker::Commerce.product
#=> "Intelligent Steel Computer"
Faker::Business.credit_card_number
#=> "1212-1221-1121-1234"
```
...

**Meryl Streep1**: Yes, that'll be one Intelligent Steel Computer, please.  Do you accept credit
cards?  
**Meryl Streep2**: So, this is the beginning of happiness. This is where it starts.
And, of course, there will always be more.  
**Meryl Streep3**: Alright ladies, that's a wrap.

