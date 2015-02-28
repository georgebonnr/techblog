---
layout: post
title: "Maker Patterns Pt. 2"
date: 2013-10-06 20:25
comments: true
published: true
categories: [javascript]
keywords: javascript,prototypal,pseudo-classical
description: all about prototypes in javascript, prototypal and pseudo-classical instantiation
---

All About Prototypes
--------------------

We've all got baggage. Javascript is no exception. It just so happens that every single function that gets defined in Javascript comes with its own companion object called a prototype. This prototype object in most cases hangs around doing nothing at all (besides creating useless overhead). When it comes time to instantiate an object in Javascript, though, this prototype "assistant" offers powerful functionality.

Before we go further with prototypes let's delve into a little backstory, which unfortunately will include discovering some of the skeletons in Javascript's closet. In 1995 Netscape tasked employee Brendan Eich with developing a new scripting language for the Netscape browser. This language would be aimed at the then-exploding class of amateur programmers ready and eager to create websites that blinked, flashed, danced, jumped, and sang. This language was also supposed to be a direct competitor to Visual Basic (readers who associate Javascript with the current world of Node, Angular, and other coolier-than-thou environments might feel... perturbed, to put it mildly, about the last two sentences.  ...I said there would be skeletons).

Under pressure to produce a working prototype of the language in a short amount of time, Brendan Eich grabbed some closures from Scheme, threw in a sweet prototype-based inheritance model inspired by Self, shrugged at or failed to notice a host of crippling idiosyncrasies (for example `alert(typeof NaN); //alerts 'Number' alert(NaN === NaN);  //evaluates false`), and called it a day.

Eich's bosses looked at the language, smiled and nodded politely, and told him to add a bunch of stuff to it to make it look more like Java. Java was the new hotness at the time and Netscape desperately wanted to leverage that hotness – enough that they even decided to name the new language Javascript, again for reasons that were mostly arbitrary and marketing-related. So the end result was a scripting language misleadingly-named Javascript that included a bastardized version of prototypal inheritance featuring some Java-esque functionality stuck on with Elmer's glue. 

Also, most of the aforementioned crippling idiosyncracies coasted through to the final version unfixed. [This](http://www.javaworld.com/javaworld/jw-03-1996/idgns.java.1995/idgns.java.1995.114.html) is what the press release looked like (note the breathless mention of "object-oriented", which was true in only the hackiest of elmers-gluiest senses, and also the embarrassing mention of visual basic).

Let's step over that skeleton clutching your ankle and move on!

If you haven't figured it out already, Protypal inheritance is the pattern that attempts to return to Eich's original ideas, and Pseudo-classical is the pattern that makes use of the fake-object-oriented parts that got added to by his bosses.

Prototypal
----------

    var Pizza = function(size) {
      var pizza = Object.create(Pizza.prototype);
      pizza.size = size;
      pizza.sauce = tomato;
      pizza.slices = 0

      return pizza;
    }

    Pizza.prototype.cheese = mozzarella;

    Pizza.prototype.slice = function() {
      this.slices += 2;
      // slice in this case is a verb, not a noun.
    }

    var myPizza = Pizza(14)

Remember I said that a prototype was a companion object. One common (and understandable) source of confusion for Javascript beginners is that when they see "Pizza.prototype" they assume it refers to some sort of parent of the Pizza constructor. In fact, to stick with the familial analogy, Pizza.prototype is more like Pizza's invisible friend that we suddenly decided to start addressing by name (perhaps this is a movie a late-80s Steven Spielberg might have directed). 

What's interesting is that the word "prototype" does in fact refer to a parent-child relationship – it means that the `Pizza.prototype` object is intended to be a parent to any pizza instances that are created using the Pizza function, such as our example `myPizza`. `Object.create()` on line 27 is what actually sets that parent-child relationship in this pattern. This means that when we refer to `myPizza.cheese`, `myPizza`, finding that it has no immediate property named cheese, will ask its designated parent object if it has that property, and if does, return that property as if it was its own. Thus `myPizza.cheese` will return `mozzarella`.

*PROS:* <br>
-    Good for memory management, since we only define the `slice` method and `cheese` property in one place and let all instantiated objects refer to those properties rather than owning their own copy (we are making the decision for now that all pizzas only need to use mozzarella cheese, which if you've ever tried to experiment with cheddar as a substitue at home you will understand). <br>
–    Cleaner and safer than using mixins (which we looked at in Pt. 1) <br>
–    Supports subclassing (which we may get into in another post) <br>
–    Makes Brendan Eich feel good about his decisions in life? <br>

*CONS:* <br>
- Worse performance than other patterns in certain scenarios. <br>
- Different than typical OOP syntax - can cause mild-to-moderate confusion for beginners coming from "classical" languages.

Pseudo-classical
----------------

    var Pizza = function(size) {

      this.size = size;
      this.sauce = tomato;
      this.slices = 0;

    }

    Pizza.prototype.cheese = mozzarella;

    Pizza.prototype.slice = function() {
      this.slices += 2;
    }

    var myPizza = new Pizza(14)

If you're comfortable with OOP this pattern will look the most familiar to you.  What may not be clear is how the pseudo-classical pattern is using prototypal inheritance under the hood. The easiest way to vizualize this is to mentally insert `this = Object.create(Pizza.prototype)` as the first line inside `Pizza` and then mentally insert `return this` as the last line in `Pizza`. This way the prototypal inheritance is established and the classical syntax is preserved (the use of the `new` keyword is what indicates to Javascript that this constructor function is intended to be used in the pseudoclassical style).

*PROS:* <br>
- Familiar to classical programmers <br>
- Has been optimized for performance in certain scenarios <br>

*CONS:* <br>
- Ridiculous bugs can happen if you forget the `new` keyword <br>
- Obscures what is really happening behind the scenes in Javascript to acheive the inheritance<br>
- Using this pattern when making subclasses (a category of constructed objects that delegates to another constructed category) can have serious downsides (I prefer a combination of this pattern with prototypal for subclassing – I might do another post on the interesting challenges that subclassing poses in js).

That's it! If you want to review the other two patterns you can find them [here](http://georgebonnr.github.io/blog/2013/10/02/maker-patterns-pt-1/).  Now go ahead and use these patterns to build your cats website. I was joking when I said it was dumb... it's gonna be great, people will love it.

As always if you have more questions feel free to email me at <georgebonnr@gmail.com>.