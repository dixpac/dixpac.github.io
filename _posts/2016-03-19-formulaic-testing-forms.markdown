---
title: "Formulaic, testing forms.Easier."
layout: post
date: 2016-03-19 12:00
tag:
- testing
- rails
blog: true
---

I've been recently playing with [thoughtbots Formulaic gem](https://github.com/thoughtbot/formulaic). I was
frustrated and bored with the way how form in rails are tested.
Luckily **Formulaic** reduced my frustration and boredom.

*Life is good again* :)

### The problem

Let's create *rails post scaffold*, for the **n** time (*n tends to infinity*).

```ruby
  bin/rails generate scaffold post title body:text
   bin/rake db:migrate
```
Ok, now let's write an integration test for adding a new post

```ruby
require 'rails_helper'

feature 'User adds a new post' do
  scenario 'successfully, when title and body is entered'  do
    visit new_post_path

    fill_in 'Title', with: 'Strange kind of women'
    fill_in 'Body', with: 'The kind that gets written down in history'
    click_on 'Create Post'

    expect(page).to have_content 'Post was successfully created'
  end
end
```

**When written on small test project like this, of course this is not too
much of a problem, but in real world rails projects consisting of hundreds of
forms this pattern of filling and exercising forms becomes pain in
ass.  
At least for me.**

Also, I like to separate my test phases(*setup, exercise, verify,
teardown*) with a new line, but writing tests like this I always ask myself *does fill form fields belong to setup or exercise phase ?*

### Let's give *Formulaic* a try!

First, in *Gemfile* add

```ruby
gem 'formulaic', group: :test
bundle install
```

also, for Formulaic to work with **rspec** add following in
*rails_helper.rb*

```ruby
RSpec.configure do |config|
  config.include Formulaic::Dsl, type: :feature
end
```

**Cool!** Time to rewrite the previous test.

```ruby
require 'rails_helper'

feature 'User adds a new post' do
  scenario 'successfully, when title and body is entered'  do
    visit new_post_path

    fill_form_and_submit(:post, 
      { title: 'Strange kind of women',
        body: 'The kind that gets written down in history' })

    expect(page).to have_content 'Post was successfully created'
  end
end
```

**Much nicer, and hey, I don't have to think in which phase filling fields belongs
to.**

If you are using fctory_girl this test becomes even more cooler:

```ruby
require 'rails_helper'

feature 'User adds a new post' do
  scenario 'successfully, when title and body is entered'  do
    visit new_post_path

    fill_form_and_submit(:post, attributes_for(:post))

    expect(page).to have_content 'Post was successfully created'
  end
end
```


## Caveats

Formulaic relies pretty heavily on the assumption that your application
is using translations for **SimpleForm** and input helpers, using the
```simple_form.labels.<model>.<attribute>``` and
```helpers.submit.<model>.<action>``` conventions.

**You can still use Formulaic by using strings as keys instead of symbols,
which it knows to pass directly to fill_in rather than trying to find a
translation. You’ll need to find submit buttons yourself since submit is
a thin wrapper around I18n.t.**

So, in our example I have to add some translations for this to work, but
hey you will do that anyway.

```ruby
helpers:
  submit:
    post:
      create: 'Create Post'
```

**Also, formulaic assumes your forms don't use AJAX, setting the wait time to 0.
This can be configured using:**

```ruby
Formulaic.default_wait_time = 5
```

If you've read this whole post, I'm sure you have irresistible desire
to listen to Deep Purple's Strange Kind of Woman.

[Indulge yourself]
[![Indulge yourself](http://img.youtube.com/vi/bAzjVdD06z8/0.jpg)](http://www.youtube.com/watch?v=YOUTUBE_VIDEO_ID_HERE)


