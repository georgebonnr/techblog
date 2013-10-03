---
layout: post
title: "Maker Patterns Pt. 1"
date: 2013-10-02 22:34
comments: true
categories: [javascript]
---

Hey kids!

Let's talk about making things in Javascript.  I don't mean your dumb cats website, I mean things IN Javascript.  What are some useful ways to make (build / instantiate) things (objects) in the Javascript language?

For this post I'm assuming that you're fairly new to Javascript, although you may have a little bit of programming experience in another language.

First of all, if you come from a language that handles object-oriented programming in a more conventional way (like Ruby or Java), you've perhaps complained to yourself that Javascript is distressingly un-pretty and complicated in its implementation of classes and inheritance.  You would be right, especially in the case that you've been trying to use Javascript in a Ruby-or-Java-like way.  However, Javascript more than makes up for this fugliness by offering significantly more power and flexibility in the ways you can create objects.  I'm going to demonstrate four different patterns for instantiating objects in Javascript. To keep things brief I'll only cover two functional patterns in this post, and in the next post I'll get into prototype chains in Javascript and prototype-based maker patterns. When it's all said and done you can choose whichever style you think will suit your project best.  If you have questions about the code used to make each example work (such as how Javascript uses the keyword `this`), you're welcome to email me with questions, or wait for an upcoming post on interesting Javascript artifacts (like `this`).

Functional
----------

    var Pizza = function(size) {
      var pizza = {};
      pizza.size = size;
      pizza.cheese = mozzarella;
      pizza.sauce = tomato;
      pizza.slices = 0
      pizza.slice = function() {
        this.slices += 2;
        // slice in this case is a verb, not a noun.
      }

      return pizza;
    }

    var myPizza = Pizza(14)

Pretty simple.  You call this function, pass in a size, and you immediately get back a pizza object (our pizza-making factory is pretty quick). <br/>
*PROS:* Simple and un-fussy. Supports functional programming style (which is a totally worthwhile thing to learn.  if you're feeling ambitious check out [this terrific article](http://msdn.microsoft.com/en-us/magazine/gg476048.aspx) on functional vs. object-oriented programming styles) <br/>
*CONS:* Each pizza comes with its own set of properties for all the things listed in the pizza object, including its own slice function (if you're wondering why that's a bad thing you will know in a second). Also, if we want to make pizzas with different types of cheese or sauce (subclasses), we'll have to buid a second equally large pizza factory (not the most efficient thing in the world, although not quite as inefficient as OOP folks would have you believe).

Functional with Mixins
----------------------

    var Pizza = function(size) {
      var pizza = {};
      pizza.size = size;
      pizza.cheese = mozzarella;
      pizza.sauce = tomato;
      pizza.slices = 0
      // this next line is a custom function we have written at the bottom of the example
      extend(pizza, slice)
      return pizza;
    }

    var slice =  {};
    slice.slice = function() {
          this.slices += 2;
    };

    var extend = function(to, from) {
      for (var property in from) {
        to[property] = from[property]
      }
    }

If you don't understand what the extend function is doing, just know that it's surveying whatever's in the slice object (in this case just a slicing function), and telling the pizza object to act like it can do whatever the slice object can do. This is called a mixin, and it's a pretty cool concept (some other languages have their own, different ways of doing mixins). You can do a lot of different things in Javascript with mixins -- if you want to investigate further, check out [this writeup](http://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/). If you're wondering if you can "mix in" the slice functionality into any object (like a pie), then you've just discovered part of why mixins are so cool. <br/>
*PROS:* We've solved the problem of each pizza having its own slicing mechanism (the problem was that each slicing function took up its own space in memory with each successive pizza. Now we have created a single slicing function that takes up only one space in memory and have told every pizza to refer to it when it needs to slice). <br/>
*CONS:* Having a bunch of random objects floating around possibly getting mixed into other objects can be a messy way to work. What if you really only wanted just pizzas to be able to slice? What if you accidentally extended your pet snake object with slice (gross)? Mixins are powerful but can be dangerous in the wrong hands.


Next Post:
I'll get into prototypes in Javascript - what they are, how they came to be, and different ways you can use them to construct objects.  If you have any questions about anything in this post you can email me through my Github profile in the sidebar. Stay tuned!