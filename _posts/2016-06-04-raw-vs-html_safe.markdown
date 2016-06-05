---
title: "Rails raw vs html_safe"
layout: post
date: 2016-06-04 13:00
tag:
- rails
- xss
blog: true
---

Rails has some neat security protections out of the box. Rails will
automatically escape html using `html_escape` method under the box,
hence preventing execution of malicious code. But, in a real world
scenario there are a lot of time where you want to render html that is
not escaped. In that scenario two of the most used methods are `raw and
html_safe`. In this article lets explore difference between this two
methods.


What is XSS(Cross-Site Scripting) ?

XSS allows an attacker to execute scripts in the security context of your web application. 
The OWASP Top 10 of most frequent vulnerabilities lists it as #3,
with other types of injection on #1 now.

You can read more about it [here](http://guides.rubyonrails.org/security.html#cross-site-scripting-xss)!

### The playground app

```ruby
rails new welcome
cd welcome
rails g controller welcome index
touch app/models/greeting.rb
```

I created basic rails app called welcome and added welcome controller
with index view.
Also, I added simple class Greeting in our models dir, this class I'm
going to use for showing basic greeting message.

OK, still a few things to setup:

1.Add root route to `config/routes.rb` and delete everything else from
class (thanks Rails 5 for removing this bloating comments in generated
files :)) 

```ruby
Rails.application.routes.draw do
  root "welcome#index"
end
```

2.Add greeting message to `Greeting` model

```ruby
class Greeting
  def message
   "Welcome jedi master"
  end
end
```

3.Add `Greeting` model instance variable to `WelcomeController`

```ruby
class WelcomeController < ApplicationController
  def index
    @welcome = Greeting.new
  end
end
```

4.Show message on the root page (`welcomes/index` page)

```erb
<p><%= @welcome.message %></p>
```

**Finally run your `rails server` got to `localhost:3000`, and you will
see following rendered on root page.**

![Basic message](http://dixpac.github.io/assets/images/plain_message.png "Basic message")

## Rendering html in view

Now lets emphasis Jedi Master title in our greeting message

```ruby
class Greeting
  def message
  "Welcome <strong>jedi master</strong>"
  end
end
```

Refresh your page and you will see that rails automatically escaped your content.

![Escaped message](http://dixpac.github.io/assets/images/escaped_content.png "Escaped message")


## I trust my content don't escape it Rails

Ok, you can used following methods to tell rails not to escape your content,
both will produce same output

in app/views/welcomes/index.html.erb

```erb
<p><%= @welcome.message.html_safe %></p>
```

or

```erb
<p><%= raw @welcome.message %></p>
```


![Raw message](http://dixpac.github.io/assets/images/raw_content.png "Raw message")

**me**:  
Look yoda master I'm giving big respect by greeting all jedis properly!

**yooda**:  
Why do you have two different methods for the same thing. Which path is
the right path to choose, young apprentice ?

### Rails raw() vs html_safe() method

**html_safe()** returns instance of `SafeBuffer` which inherits from
`String` overriding `+`, `concat` and `<<` so that:

* If the string is safe (another `SafeBuffer`), 
  the buffer concatenates it directly
* If the string is unsafe (a plain `String`), 
the buffer escapes it first, then concatenates it

You can look at the implementation
[here](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/string/output_safety.rb#L135)


**raw()** is a wrapper around html_safe() that forces the input to String and 
 then calls html_safe() on it. It’s also the case that raw() is a helper in a module

Basically, you can simulate raw in out example like this

```ruby
class Greeting
  def message
    "Welcome <strong>jedi master</strong>".to_s.html_safe
  end
end
```

The main reason to use `raw` instead of `html_safe` if when content can be `nil`.

If you call ```html_safe``` on `Nill` class, like this  

```ruby
nil.html_safe
```
app fill fail saying ```undefined method `html_safe' for nil:NilClass```.

But if you call `raw` on `Nil` class, app will not fail it will return
instance of  empty string, since calling ```nil.to_s``` returns new
empty_string `""`

## Summary

* If a plain String is passed into a <%= %>, Rails always escapes it
* If a SafeBuffer is passed into a <%= %>, Rails does not escape it.
To get a SafeBuffer from a String, call html_safe on it.
The XSS system has a very small performance impact on this case,
limited to a guard calling the html_safe? method
* If you use the raw helper in a <%= %>, Rails detects it at compile-time of
the template, resulting in zero performance impact from the XSS system on that concatenation
* Rails does not escape any part of a template that is not in an ERB tag.
Because Rails handles this at template compile-time, this results in zero performance impact from the XSS system on these concatenations
* If you have string which can be nil consider using `raw()` because
`html_safe()` will throw an error


**Be carefull not to let sits inject malicouse dark force to young
jedis**
