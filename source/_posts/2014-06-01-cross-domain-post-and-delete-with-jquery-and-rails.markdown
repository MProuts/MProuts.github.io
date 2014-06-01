---
layout: post
title: "Cross-Domain Requests With Angular & Rails"
date: 2014-06-01 09:54:29 -0400
comments: true
categories:
---

Most modern web APIs provide programatic access to an application's internals by exposing RESTful JSON endpoints.  For example, a ```POST``` to ```/pokemon.json``` with a lump of properly-formatted JSON might create a new pokemon resource on the application server.  

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
  headers['Access-Control-Request-Method'] = '*'
end
```


#POST
 ```POST``` should now also work.

```js
$.ajax({
  type: "post",
  url: 'http://localhost:3000/pokemon/' + pokemon.id + '/comments';
  data: mydata,
  dataType: "json",
  success: function(data){
    ...
  }
});
```
#DELETE

 ```GET``` and ```POST``` are considered 'simple requests' under the
 CORS standard, meaning that providing the access control headers is
 all we have to do.

What about ```DELETE```?

###app.js

```js
$.ajax({
  url: 'http://localhost:3000/pokemon/' + pokemon.id + '/comments/' + comment.id;
  dataType: "json",
  type: "DELETE",
  contentType: "application/json",
});
```

...produces...

{% img /images/cors_delete_request.png %}

...and in the Rails log...

    Started OPTIONS "/pokemon/11/comments/119" for 127.0.0.1 at 2014-06-01 11:38:14 -0400

    ActionController::RoutingError (No route matches [OPTIONS] "/pokemon/11/comments/119"):

#Preflighted Requests
Requests that use methods other than ```GET``` or ```POST```, or that
use custom headers, need to be 'preflighted'.  Essentially, before the
actual request can take place, the client sends a preflight
 ```OPTIONS``` request with an ```access-control-request-method```
header, asking permission to use a particular method.  In our case, we
want to ```DELETE```.  jQuery is smart enough to initiate the preflight
request for us.

    OPTIONS /pokemon/11/comments/119
    Origin: http://localhost:8000
    Access-Control-Request-Method: DELETE

If the server responds with

    Access-Control-Allow-Origin: http://localhost:8000
    Access-Control-Allow-Methods: DELETE

...then the browser allows the original request go through.

Rather than roll our own CORS routes and logic for handling preflighted OPTIONS requests and headers,
we can simply add the ```rack-cors``` gem.

###Gemfile
```ruby
  gem 'rack-cors', :require => 'rack/cors'
```

###config/application.rb
```ruby
config.middleware.use Rack::Cors do
  allow do
    origins 'http://localhost:8000'
    resource %r{/pokemon/\d+/comments/\d+},
      :headers => ['Origin', 'Accept', 'Content-Type'],
      :methods => [:delete]
  end
end
```

This will intercept and handle the OPTIONS request and grant DELETE
access to ```localhost:8000```.

And there you have it.
