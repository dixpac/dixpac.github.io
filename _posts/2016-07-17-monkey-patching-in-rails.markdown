---
title: "Monkey patching in Rails"
layout: post
date: 2016-07-16 13:00
tag:
- rails
- ruby
blog: true
---

[Monkey Patching.](https://en.wikipedia.org/wiki/Monkey_patch) It’s amazing. One of the most powerful Ruby features.
You have ability to open any ruby class and change how it works, add new
methods, basically you can do almost anything. But, as we all now from
the great power comes great responsibility, and putting the power in the
human hands is usually not so great idea.

![Monkey with a gun](http://dixpac.github.io/assets/images/monkey-gun.jpg "Monkey patcher")

Let's see how to use Monkey Patching in Rails, where to store the code,
and how to make you life easier in 1 year from now when things stop to
work because of the patch you/someone wrote.

### The problem and example app

Surprisingly, 90% of Rails application I was involved in required
ability to check if `String` value is a `Number`. Let's say we have `analyses`
table which stores name, and the value of analysis done on the blood sample. 

Some of the values can be `Number`(ex: `1.5, 2.45, etc..`) some of
them just plain `String`(`positive, negative, etc...`). But we will go
easiest route in this example and store each result in ```value``` column
which is ```String```. 

Let's roll!

```ruby
rails new sampler # create new rails application
bin/rails g model analysis name value # create analysis model
bin/rake db:migrate # create the table
```

Now, insert some test data into the analyses table:

```ruby
bin/rails console
Analysis.create(name: "Leukocite", value: "7.56")
Analysis.create(name: "Blood color", value: "blue")  # watch it vampire!!!
```

Since in this fictive application I need ability to figure out if the
value of Analysis is string or number, I need a way to check if `String`
is valid `Number`. I will open `String` class and add method `is_number?` Which
will check if string is number (**NOTE:** *not the only and certainly not the
fastest way to resolve the problem but for the sake of this post, I will do
it*).


```ruby
class String
  def is_number?
    true if Float(self) rescue false
  end
end
```

Now I can call `is_number?` On each string value in my app,like this

```ruby
Analysis.first.value.is_number?  ⇒ true
Analysis.last.value.is_number?   ⇒ false
"ok".is_number?                  ⇒ false
```

**Well, not so fast! How to add this in Rails app and where to add it
?**

### Put them in a module 

When you monkey patch a class, don’t just reopen the class and shove
your patch into it:

```ruby
class String
  def is_number?
    true if Float(string) rescue false
  end
end
```

Why not?

* If two libraries monkey-patch the same method, you won’t be able to tell.
* If there’s an error, it’ll look like the error happened inside
  String(*While technically true, it’s not that helpful*).
* It’s harder to turn off your monkey patches.
* If you, say, forgot to require 'string' before running this monkey
  patch, you’ll accidentally redefine `String` instead of patching it.

Instead, put monkey patches in a module:

```ruby
module CoreExtensions
  module String
    module Number
      def is_number?
        true if Float(self) rescue false
      end
    end
  end
end
```
This way, you can organize related monkey patches together. When there’s an error, it’s
clear exactly where the problem code came from.
And you can include them one group at a time:

```ruby
# Actually monkey-patch String, you can add this to Rails initializers
String.include CoreExtensions::String::Number
```

### Keep them togeather & ogranized

When you monkey patch core Ruby classes you are add/change Ruby API. You
need a way to quickly find and learn those changes when you jump in
codebase.

I mostly follow Rails’ monkey patching convention. Patches go into 
```lib/core_extensions/class_name/group.rb``` or
```lib/core_ext/class_name/group.rb```.

So this patch:

```ruby
module CoreExtensions
  module String
    module Number
      def is_number?
        true if Float(self) rescue false
      end
    end
  end
end
```
will go into:
```lib/core_extensions/string/number.rb```


**Now you/new developers can easily jump in lib/core_extensions and see 
all of the monkey patching adventures :)**


### Conclusion

Always think through when you want to do monkey
patching, is there better way. In this example
definitely there is, we could store results
differently not everything in one string column. But
this is just the naive example :)

**If you go for the big guns it is nice to know how to use them.**
