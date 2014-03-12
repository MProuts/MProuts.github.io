---
layout: post
title: "Bootstrapping a Sinatra App"
date: 2014-03-10 12:00:09 -0400
comments: true
categories: 
---

Sinatra is a simple open-source web framework for ruby.  It's similar to Rails,
but much smaller, weighing in around 1,500 lines of code compared to Rail's
~100,000.  

Its stripped-down nature means that out of the box, Sinatra does less *for* you (automatically) than does Rails.  For example, if you tell Sinatra to ```generate``` a ```scaffold``` for you, it will respond politely that it has no idea what the [Sinatra] you're talking about.  What's more, much of the nitty-gritty of configuring you're environment (such as managing your environmental dependencies and database connections) is left up to you, the programmer.  

Thus, working with Sinatra, you pay a higher setup cost upfront (than in Rails), 
in exchange for a much smaller codebase, and a better understanding of how your
application does everything that it does. The following is aimed at minimizing
that cost while maximizing understanding.

Ready?  Here goes:

1) Database/ActiveRecord
=====================

This example uses sqlite with ActiveRecord.  As mentioned, in Sinatra, you're in charge of establishing and managing your database connection manually.  It’s convention to put this, along with all of your other environmental dependencies in a file called ```environment.rb``` in your project’s ```config/``` directory.
###Gemfile
```ruby
source "https://rubygems.org"

gem "sinatra"             
gem "sqlite3"               
gem "activerecord"          
gem "sinatra-activerecord" 
```
###config/environment.rb
```
#Set SINATRA_ENV to development if not already set
ENV['SINATRA_ENV'] ||= "development"

require 'bundler/setup'
Bundler.require(:default, ENV['SINATRA_ENV'])

ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/project_#{ENV['SINATRA_ENV']}.sqlite"
)
```
As you can see, our ```environment.rb``` requires our bundled gems and sets our database connection conditionally based on our current environment (which we’ve stored in an environment variable).

###controllers/application_controller.rb

```
class ApplicationController
  register Sinatra::ActiveRecordExtension
  ...
end
```

###Rakefile
```
require './config/environment'
require 'sinatra/activerecord/rake'

namespace :db do
  task :down do
    FileUtils.rm('./db/development.sqlite')
  end

  task :up do
    Rake::Task["db:migrate"].invoke
    Rake::Task["db:seed"].invoke
  end
end
```
Now that we’ve taken care of connecting to our database, we can think about interacting with it.  The ```sinatra-activerecord``` gem provides familiar database interface in the form of predefined Rake tasks like ```rake db:migrate```  and ```rake db:rollback```.  To use it, we include it in our Gemfile, register it in our ApplicationController, and require it in our Rakefile.  

In addition to the tasks provided by ```sinatra-activerecord```, I wrote two custom rake tasks — one for deleting the entire database, and one for migrating and re-seeding it.  

###config.ru
```

...
if defined?(ActiveRecord::Migrator) && ActiveRecord::Migrator.needs_migration?
  raise 'Migrations are pending run `rake db:migrate` to resolve the issue.'
end
...
```

###spec/spec_helper.rb
```
ENV['SINATRA_ENV'] = "test"

if ActiveRecord::Migrator.needs_migration?
  raise 'Migrations are pending. Run `rake db:migrate SINATRA_ENV=test` to resolve the issue.'
end
```

Finally, we raise helpful reminders in ```config.ru``` and our ```spec_helper.rb``` if either gets run when there are still pending migrations.  

2) Console, Local Server
=======================
###Gemfile
```
...

group :development do
  gem "pry"
  gem "shotgun" 
end
```

###Rakefile
```
task :environment do
  require './config/environment'
end

task :console => :environment do
  Pry.start
end
```
In addition to managing our database conneciton, we’ll have to roll own console and server tasks in Sinatra as well.  We could rackup directly from our config.ru, however, this would mean stopping and starting the development server any time we made any changes to our application.
Instead, we use a gem called shotgun, which takes care of reloading for us.  After installing type ```shotgun``` to fire up the server.

Pry provides a feature-rich debugging shell.  To use it as our console we simply define a rake task that requires our environment and starts Pry.

3) MVC Structure
===============
Sinatra lets you place your model and controller code side-by-side in a general application file.   (It  will look for views in a top-level ```views/``` directory by default. This may be a good option for you, depending on the size and complexity of your project.  However, if your requirements necessitate breaking your application up into a more Railsy MVC structure, here’s how to do it.  


----------------------
###controllers/application_controller.rb
```
class ApplicationController < Sinatra::Base
end

```
(1) Define an application controller that inherits from ```Sinatra::Base```.  
(2) Mount it as middlewear in ```config.ru```.  
(3) Define individual resource controllers that inherit from ```ApplicationController```.  
(4) Mount them as middle-wear on top of ```ApplicationController```.   

###config.ru

```
...
use UsersController
use OutfitsController
use ArticlesController
run ApplicationController
```
--------------------
(5) Move your views to ```app/views```.  
(6) Move your models to ```app/models```.

###app/controllers/application_controller.rb
```
class ApplicationController < Sinatra::Base
  set :views, Proc.new { File.join(root, "../views/") }
  set :method_override, true
  set :sessions, true
end
```
(7) Tell Sinatra about the new location of your views.  
(8) Turn on method_override if you want to use RESTful HTTP verbs via the hidden input hack.  
(9) Turn on sessions if you plan to use them.

-------------------
###config.ru
```
use Rack::Static, :root => 'public', :urls => ["/images","/javascripts","/stylesheets"]
```
(10) Tell Sinatra about the new location of your static resources.  

---------------------
###Gemfile

```
...
gem "require_all" 
```

###config/environment.rb
```
...
require_all 'app'
```
Now that our application code is split across many files within or ```app/``` directory, it’d be nice to have a way to require all of those files in one fell swoop.  Luckily, that’s exactly what the ```require_all``` gem lets us do.  Just include it your ```Gemfile``` and ```require_all``` your application directory (conventionally ‘app’) in your ```environment.rb```.
4) Testing
========
###Gemfile
```
...
group :test do 
  gem 'database_cleaner', git: 'https://github.com/bmabey/database_cleaner.git'
  gem "rspec"
  gem "rack-test"
  gem "pry"
end
```
To get up and running with testing we’ll put ```rspec``` and a few other gems in our gemfile.  After ```bundling``` and running ```rspec --init```, open your ```spec_helper``` and include Rack::Test::Methods in the configuration block to get access to rack::test stub methods.  

E.g.
```
post ‘/students’, { “name” => “Michael” }
``` in your controller specs.  

###spec/spec_helper.rb
```
require_relative '../config/environment'

RSpec.configure do |config|
  ...
  config.include Rack::Test::Methods

  config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end
end
```
Another useful tool is the DatabaseCleaner, which resets your database between examples in your tests to ensure your tests are database-independent.

Phew.  So there you have it -- a basic bootstrap for a MVC app in Sinatra.  Enjoy, and don’t spend it all in one place.  ;)   
