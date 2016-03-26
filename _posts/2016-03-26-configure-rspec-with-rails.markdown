---
title: "Configure rspec to work with rails."
layout: post
date: 2016-03-19 12:00
tag:
- testing
- rails
blog: true
---

Lots of people tend to use RSpec when working with rails instead default
MiniTest. RSpec isn't included in a default Rails application, and
there is boring process configuring RSpec if you are RSpec user, unless
you have configured some custom templates or you are using some custom
Rails app generators. [Suspenders](https://github.com/thoughtbot/suspenders) is great example of custom generator
which come with Rspec configured by default.

Lets assume that you want to configure RSpec from ground up, and there
is no any custom generators magic included :)

Creating the new app:

```ruby
rails new cool_app -T
```

**-T** option tells default Rails generator to skip creating test directory.
**If you already generated app with MiniTest, just delete test directory
from Rails app and your good to go.**

## Installing gems:

```ruby
group :development, :test do
  gem 'byebug'
  gem "rspec-rails"
end

group :test do
  gem "capybara"
  gem "database_cleaner"
  gem "selenium-webdriver"
end
```
rspec-rails is used in both the development and test environments. 
Specifically, they are used in development by generators we’ll be utilizing
shortly. The remaining gems are only used when you actually run your specs, so
they’re not necessary to load in development. This also ensures that gems used solely
for generating code or running tests aren’t installed in your production environment
when you deploy to your server.

Run ```bundle install``` from command line to install newly added gems.

What are these gems that we added to the gem file:

* [rspec-rails](https://github.com/rspec/rspec-rails) includes RSpec itself in a wrapper to add some extra Rails-specific features.

* [capybara](http://jnicklas.github.io/capybara/) makes it easy to programatically simulate your users’ interactions
with your web application.

* [database_cleaner](https://github.com/DatabaseCleaner/database_cleaner) helps make sure each spec run in RSpec begins with a clean slate, by–you guessed it–cleaning data from the test database.

* [selenium-webdriver](https://github.com/seleniumhq/selenium) will let us test JavaScript-based browser interactions with Capybara.

## RSpec configuration:

To add some basic RSpec configuration run following command line directive

```ruby
bin/rails generate rspec:install
```
*You should see following console output*

```
create .rspec
create spec
create spec/spec_helper.rb
create spec/rails_helper.rb
```

Rspec creates *.rspec* configuration file, *spec/* directory 
where all new specs are going to live, and two helper files
*spec_helper* and *rails_helper* where we will further customize
how RSpec will interact with our code.

Next change RSpec’s output from the default format to the easy-to-read 
documentation format. This makes it easier to see which specs are
passing and which are failing as your suite runs. It also provides an attractive outline
of your specs for–you guessed it–documentation purposes. Open *.rspec* and add the
following lines:

```
--format documentation
```

Finally, let’s install a binstub for RSpec:

```
bundle binstubs rspec-core
```

Now when you run ```bin/rspec``` or ```bin/rake```, your sould see message
simillar to this in console output:

```
0 examples, 0 faliures
```

Cool rspec is ready to roll !!

## Cleaning database:

Now everythings works fine, but here is the problem, **if you create 
some new database rows in one spec, you test database is not going to clean/destroy
those rows after spec finishes**, therfore violating one of the 4 main testing phases:

* **Setup** the system under test (usually a class, object, or method) is set up.
* **Exercise** the system under test is executed.
* **Verify** the result of the exercise is verified against the developer’s expectations.
* **Teardown** the system under test is reset to its pre-setup state.

*Main problem is not violating Teardown pahse, but the living hell of problems
you will get into if there is no some sort of databse cleaning strategy.*

**database_cleaner gem to the rescue!**

Easilly we will setup stategy that cleans database after every spec, so we begin each spec
with clean slate database.

Inside **spec/** directory create directory **support/** and add **database_cleaner.rb** file with
following content:

```ruby
RSpec.configure do |config|
  config.before(:suite) do
    DatabaseCleaner.clean_with(:deletion)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, js: true) do
    DatabaseCleaner.strategy = :deletion
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
```
also, ```require "databse_cleaner"``` in *rails_helper* and uncomment 
this line of code (for RSpec to be able to pick support files):

```ruby
Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }
```

There is a lot of going on in the *database_cleaner* setup which is out of the scope
for this post, you should read more [here](https://github.com/DatabaseCleaner/database_cleaner/blob/master/README.markdown).

## Testing javascript interaction:

If you have test with javascript interaction (usually ingeration/feature) tests,
just add ```js: true``` option to yout spec (that is why I added **selenium-webdriver** in Gem file),
as in following example:

```ruby
require "rails_helper"

scenario "Some cool scenario" do
 feature "includes JS magic", js: true do
   # do your magic here
 end
end
```

Happy testing, here is the [demo repo](https://github.com/dixpac/cool_app)!
