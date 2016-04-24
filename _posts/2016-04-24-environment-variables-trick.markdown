---
title: "Work with environment varaibles easier in Rails"
layout: post
date: 2016-04-24 12:40
tag:
- rails
blog: true
---

Face it, if you are working with real world rails project you have a
lot of environment variables. Third party service keys, database, etc...   
This was always pain for me because I had to set this through
terminal (*I always forget process how to do this, so I need to spend
some time on google to check how to set ENV variables in specific OS*).

Recently I came a cross a very sexy technique for setting ENV variables
in you rails app, and process is quite **trivial**.

Add `local_env.yml` file to rails `config/` dir   
`touch config/local_env.yml`

in this yml file add all of yours ENV variables, for example

```
AWS_ACCESS_KEY: '<placeholder>'
STRIPE_KEY: '<placeholder>'
SLACK_KEY: '<placeholder>'
```

Next step  is to load all these yml keys/value pairs into ENV variables when your rails app
is starting. 

Add following in `config/application.rb`

```ruby
config.before_configuration do
  env_file = File.join(Rails.root, "config", "local_env.yml")
  YAML.load(File.open(env_file)).each do |key, value|
    ENV[key.to_s] = value
  end if File.exist?(env_file)
end
```

**And voalla...magic....you are handling all of you app specific ENV
variables through simple rails yml file.**

**NOTE**: you will also want to add this file to `.gitignore`.

If you need some more complex ENV variable manipulation you can also take a
look at [Figaro gem](https://github.com/laserlemon/figaro).
