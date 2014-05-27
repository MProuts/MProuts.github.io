---
layout: post
title: "Build A (tiny) RESTful API with Rails"
date: 2014-05-26 15:53:40 -0400
comments: true
categories: Angular, Rails, API
---

jTraditionally, APIs were public-facing interfaces -- they were for getting access to an application's data
from *outside* of the application.  With the advent of front-end frameworks like Angular,
there's been a trend toward using APIs internally as well -- building
front-end applications that consume their own internal APIs.

In this post I will show you how I made a very tiny API (just a snack) in Rails with an Angular application on top of it.

###application_controller.rb

```ruby
  ...
  protect_from_forgery with: :null_session
  ...
```

Our API will need to serve ajax requests without authenticity tokens, so
we change the default CSRF-protection setting.

#####CORS
To make the two entities as autonomous as possible, I decided to host the Angular application outside of the Rails
application.  In doing so, I encountered a number of practical issues
related to cross-domain requests.

Most browsers implement the *same-origin policy*, meaning that for
security reasons, they prevent scripts from making requests for resources that live at other domains.

Cross-Origin Resource Sharing (or CORS) is a mechanism for getting
around this restriction.  Basically, the requesting script and the
server use special headers to convince the browser they know each other
before the request is processed. If the requesting script's domain is on 
the guest list returned in a special header by the server, the browser will allow the request.
