---
title: "Depend on Behavior, Not Data"
layout: post
date: 2016-04-10 16:00
tag:
- clean-code
- ruby
blog: true
---

Since most of us write object-oriented code, we are creating classes a lot of classes.

Why?   

We need place to store behavior and data
that belongs together. Hence the **classes**! 
Nothing special but behavior and data together on "one" place.

**Behavior** is captured in methods and invoked by sending messages. When you create classes that have a single responsibility, every tiny bit of behavior lives in one and only one place. The phrase *“Don’t Repeat Yourself ”* (DRY) is a shortcut for this idea. DRY code tolerates change because any change in behavior can be made by changing code in just one place. 

In addition to behavior, objects often contain data. **Data** is held in an instance variable and can be anything from a simple string or a complex hash. Data can be accessed in one of two ways; you can refer directly to the instance variable or you can wrap the instance variable in an accessor method.

Let us create a class that represents **Person**. In our app we want Person responding to bmi(Body mass index) message.  

*Body mass index (BMI) is a measure of body fat based on height and weight that applies to adult men and women.*

Formula for calculating BMI is:

```
BMI = ( Weight in Kilograms / ( Height in Meters x Height in Meters ) )
```

{% gist dixpac/3350c9fa24ad9168340385a795d45fa7 %}

Now we can use calculate Person BMI easily

```ruby
curry = Person.new(1.90, 85)
curry.bmi ====> 23.5
```

## Hide instance variables
Always wrap instance variables in accessor methods instead of directly referring to variables. Hide the variables, even from the class that defines them, by wrapping them in methods. 
Ruby provides ```attr_reader``` as an easy way to create the encapsulating methods: 

{% gist dixpac/8affcc16fc016fbbffbd0e74c7607105 %}

Using attr_reader caused Ruby to create simple wrapper methods for the
variables. Here’s a virtual representation of the one it created for
weight:

{% gist dixpac/d61f6a9cf984bd24e814eb781700f4a4 %}


This ```weight``` method is now the only place in the code that understands what
```weight``` means. Weight becomes the result of a message send. 
Implementing this method changes weight from data (which is referenced all over) to behavior
(which is defined once).  

If the *@weight* instance variable is referred to ten times and it suddenly
needs to be adjusted, the code will need many changes. However, if
@weight is wrapped in a method, you can change what weight means by implementing
your own version of the method. 
Your new method might be smart and figure out that everybody lies about their weight :)

```ruby
  # default implementation via attr_reader
  def weight
    weight * expected_lie_factor
  end
```

This example is arguably trivial and could  have been done by making one change to
the value of the instance variable. However, you can never be sure that
you won’t eventually need something more complex (*our methods is not
that smart, right now*). But this adjustment is a simple behavior change when done in a method, but a code
destroying mess when applied to a bunch of instance variable references.

***You should hide data from yourself. Doing so protects the code from being affected
by unexpected changes. Data very often has behavior that you don’t yet
know about. Send messages to access variables, even if you think of them
as data.***
