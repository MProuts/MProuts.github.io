---
layout: post
title: "Cross-Domain Requests: Angular & Rails"
date: 2014-06-01 09:54:29 -0400
comments: true
categories:
---

Most modern web APIs provide programatic access to an application's internals by exposing RESTful JSON endpoints.  For example, a ```POST``` to ```/pokemon.json``` with a lump of properly-formatted JSON as a parameter might create a new pokemon resource on the application server.  

Javascript frameworks like Angular were designed to
sit on top of APIs, accessing data programatically from endpoints like
the one above. However, this process is less straightforward than it
might be because of a restriction that most browsers implement
called the ```same-origin policy```, whereby scripts using ```XMLHttpRequest``` aren't allowed to access/manipulate resources across domains.

This means, essentially, that unless your frontend Angular application is hosted on the same
domain as the API it consumes, you will run into issues when trying to
access data.  

Luckily, most browsers also provide a mechanism for getting
around this restriction called Cross-Domain Resource Sharing or CORS.  Here's
what you need to know to get it working with jQuery and Rails.

#GET
Let's say that in development, you have your Angular app up [on port 8000](#)
and your Rails API up on port 3000. When your Angular app loads, it will probably need to reqeust some data
from the server...

###app.js
```js

$http
  .get('http://localhost:3000/pokemon.json')
  .success(function(data){
    pokedex.pokemons = data;
});
```
...but then...

{% img /images/cors_get_request.png %}

Under the same origin policy, the browser will block cross-domain
XMLHttpRequests unless the server's response includes a ```access-control-allow-origin``` header that matches the
script's request's ```origin``` header (i.e. the script's domain).

In Rails, we can simply add this header to all responses (since we
want to give the script access to the entire API) in our
application controller as follows.

###application_controller.rb
```ruby
before_filter :set_access_control_headers

def set_access_control_headers
  headers['Access-Control-Allow-Origin'] = 'http://localhost:8000'
  headers['Access-Control-Request-Method'] = 'http://localhost:8000'
end
```

#POST

In another controller, our Angular application posts data to the
server.

```js


###CORS
