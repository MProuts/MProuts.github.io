---
layout: post
title: "Friendly Forwarding With Devise"
date: 2014-04-06 17:10:43 -0400
comments: true
categories: 
---

It's pretty common to require that a user be logged in to perform certain
actions around your application. For example, let's say you wanted to make sure a user
was
signed in before being able to see any user's profile. Assuming you're using Devise for
authentication, your code might look something like this:

###controllers/users_controller.rb
```ruby

class UsersController < ApplicationController
  before_action :set_user, :only => [:show]
  before_action :require_login, :only => [:show]

  def show
  end

  private
  def set_user
    @user = User.find(params[:id])
  end

  def require_login
    redirect_to new_user_session_path, :notice => "Please sign in."  unless user_signed_in?
  end
end

```

Here, our before action, ```require_login```, conditionally redirects to the login page if a request hits the ```user#show``` action and no user is signed in.
Note that we make use of Devise's handy ```user_signed_in?``` method to make this happen.

Cool.

#Now what?

But what will the user see after they log in?  In other words: 

If I...

   1) browse to a restricted page,  
   2) get redirected to login page,  
   3) sign in,  
   4) ???  

...then where do I go next?

Out of the box, Devise redirects to the ```root_url``` upon successfully signing in a
user.  In certain cases, this may make sense from the perspective of the
user.  However, in the case outlined above, redirecting to the ```root_url``` breaks
the deal our application struck with the user in step 2. 

Here are the same four lines in plain English:

1) User: "Hey application, show me this thing."  
2) Application: "Hey user, I will, as long as you log in first."  
3) User: "Totes, np!" \<logs in cheerfully\>   
4) Application: "Here's the homepage, what can I help you with?"  

Absurd, I say.  Absurd.

#Forward!

So we need to make our application less rude by giving it a tiny bit of short-term
memory, but how can we accomplish this in a stateless protocol like HTTP?

Answer: Sessions.

A session (or "user session") is a mechanism for persisting bits of
user-specific data across multiple HTTP request cycles, giving the illusion that
an application *maintains state*, or remembers things about its past interactions with a particular user. 


How exactly sessions are implemented is beyond the scope of this post, but for now, we'll content ourselves with the fact
that we can stick information into a hash-like variable called ```session``` in our
controllers, and then retreive it when handling subsequent requests.

Back to our example.  In order to keep track of which page the user was asking
for
when we told them to sign in, we can jot down a reminder for ourselves in the ```session```.  

###controllers/users_controller.rb
```
private
def require_login
  unless user_signed_in?
    session[:forward_url] = request.fullpath
    redirect_to new_user_session_path, :notice => "Please sign in."
  end
end

```

That way, instead of trying to remember our entire conversation, (which is, well, impossible), 
we simply check the session for reminders
everytime we log a user in. 

###controllers/application_controller.rb

```
  def after_sign_in_path_for user
    session[:forward_url] ? session.delete(:forward_url) : super
  end
```

Devise calls a method aptly name ```after_sign_in_path_for``` to determine where to send the client after login.
This method is implemented internally in Devise, but we can override it by *duck
punching* it in our ```ApplicationController```.

Our implentation checks for a reminder in the session, deletes and returns that
reminder if
it exists, and otherwise passes the call up the inheritance tree to Devise's
default ```after_sign_in_path_for``` method.

