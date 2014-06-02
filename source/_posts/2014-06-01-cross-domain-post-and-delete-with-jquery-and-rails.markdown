---
layout: post
title: "Cross-Domain Requests With Angular & Rails"
date: 2014-06-01 09:54:29 -0400
comments: true
categories:
---

Most modern web APIs provide programatic access to an application's internals by exposing RESTful JSON endpoints.  For example, a ```POST``` to ```/pokemon.json``` with a lump of properly-formatted JSON might create a new pokemon resource on the application server.  

Javascript frameworks like Angular were designed to
sit on top of APIs, accessing data through endpoints like
the one described above. When an Angular app and the API it consumes are
hosted on separate domains, data access is complicated by a security restriction implemented by most browsers called the ```same-origin policy```.  Under the ```same-origin policy```, client-side scripts running in one domain are prevented from obtaining data retrieved from another domain.

Luckily, most browsers also provide a mechanism for getting
around this restriction called Cross-Origin Resource Sharing, or CORS.  Here's
what you need to know to get it working with jQuery and Rails.

#GET
Let's say that you have your Angular app running in development on port 8000,
and your Rails API running on port 3000. When your Angular app loads, it will probably need to request some data
from the server.

###app.js
```js

$http
  .get('http://localhost:3000/pokemon.json')
  .success(function(data){
    pokedex.pokemons = data;
});
```

But this produces an error.

{% img /images/cors_get_request.png %}

Under the ```same-origin policy```, the browser will block cross-domain ```XMLHttpRequests``` unless the server's response to such requests includes an ```access-control-allow-origin``` header that matches the
requesting script's ```origin``` (In our case, ```http://localhost:8000```).

In Rails, we can simply add this header to all responses
in our application controller as follows.

###application_controller.rb
```ruby
before_filter :set_access_control_headers

def set_access_control_headers
  headers['Access-Control-Allow-Origin'] = 'http://localhost:8000'
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

 ```GET``` and ```POST``` are considered *simple requests* under the
 CORS standard, meaning that providing the access control headers is
 sufficient for cross-domain requests.

But what about ```DELETE```?

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
Under CORS, requests using methods other than ```GET``` or ```POST``` need to be *preflighted*.  Essentially, before the
actual request can take place, the client sends a preliminary
 ```OPTIONS``` request with an ```access-control-request-method```
header, asking permission to use a particular HTTP method with a given URL.  In our case, we
want to use ```DELETE``` at ```/pokemon/11/comments/119```.  jQuery is smart enough to initiate the preflight
request for us...

    OPTIONS /pokemon/11/comments/119
    Origin: http://localhost:8000
    Access-Control-Request-Method: DELETE

...so if the server responds with...

    Access-Control-Allow-Origin: http://localhost:8000
    Access-Control-Allow-Methods: DELETE

...then the browser will allow the original request to go through.

Rather than roll our own system for handling this exchange,
we can simply add the ```rack-cors``` gem to our gemfile.

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

As configured above, ```rack-cors``` will intercept the preflight OPTIONS request from jQuery and grant DELETE access to ```localhost:8000```.

And there you have it.  :)
