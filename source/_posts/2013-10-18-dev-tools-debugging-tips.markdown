---
layout: post
title: "Dev Tools Debugging Tips"
date: 2013-10-18 09:58
comments: true
categories: 
keywords: 
description: 
---
NOTE: This is a blog post I wrote for incoming students at Hack Reactor -- Hack Reactor ended up liking it so much they made it part of the pre-course curriculum! If you're a new or prospective developer, hopefully you'll find it helpful too.

As you might imagine we use Chrome Dev Tools quite a bit at Hack Reactor.  Dev Tools has quickly become an indispensable tool for front-end work, but it's even more valuable as a debugging tool for your javascript programs.

If you don't already use it in your debugging workflow (I'm looking at you, chronic console.logger), approximately NOW would be a good time to learn.  I'm going to use a simple bug scenario to introduce you to Dev Tools debugging.

Imagine we are given a problem to solve:

    // given an array, write a function that will, for every item in the array, 
    // alert to the screen a string containing the item and the item's index in the array.
    // you have been given a "loop" function, which takes an array and also 
    // takes a function which will be called on every item in the array.

    var example = ["a","b","c"];

    var loop = function(array, iterator) {
      for (var i=0; i<array.length; i++) {
        iterator(array[i]);
      }
    };
our solution:
    var alertArray = function(array) {
      loop(array, function(item, index){
        alert(item + " " + index);
      });
    };

So let's run this solution.  I set up a simple html page and paste the code we wrote in a script.  I'm going to invoke the function we wrote by adding `alertArray(example);` to my script.


When I open the page and the script runs I see:

<img src="{{ root_url }}/images/devtools_post/alert1.png" />

<img src="{{ root_url }}/images/devtools_post/alert2.png" />

<img src="{{ root_url }}/images/devtools_post/alert3.png" />

Ruh-roh.  Where's the index?  You probably already know why it's not displaying, but I'm going to use this simple scenario to demonstrate what a typical debugging workflow would look like.  A good question to ask yourself when you encounter a bug is "what is the justification for my expectation that my code will run the way that I expect?

If you think you can identify a place where your expectation and the reality of the code (the bug) might diverge, enter a new line above that point and type `debugger;`  This will cause Chrome to pause execution at that line and wait for further instructions to proceed.

If you prefer not to type anything in your code you can set virtual debugger statements called breakpoints in Dev Tools – one downside of breakpoints, though, is that if you change the number of lines in your code and run the script again, Chrome's breakpoints may wind up on lines that you didn't intend.
Here's what happens in Dev Tools (`command`+`option`+`i` in Chrome) when we run the code again with our debugger statement inserted:

(open images in new tab to see them at full size)
<img src="{{ root_url }}/images/devtools_post/debug1.png" />

Step over/into/out of are very handy tools that you will be using for potentially hours on end, so here are some more details on each:

- step over will proceed by a single line of code. If there is a function call in the executed line, step over will call the function and do whatever work is contained therein... but you will not see the execution of the function. Useful if you need to proceed through a function call that you are sure is not the source of your bug.

- step into will also proceed by one line of code. If there is a function call it will transport you to the place in your code where that function call is actually running.  You will be able to inspect the internals (variables, etc) of that called function.

- step out of does not "rewind" out of a function call (which the arrow pointing up may make you think at first), rather it will "fast forward" through the rest of a function (stepping over any other function calls contained therein) and return you to its parent function.

So to proceed I'm going to click step over (or step into) to get to the beginning of line 18 (in the context of line 17 step over and step into will do the same thing since there's no function call on that line).

At the beginning of line 18 my console will look exactly the same.  Line 18, however, contains the call to the loop function, so I'm going to click "step into."  Here's what I'll see:

<img src="{{ root_url }}/images/devtools_post/debug2.png" />

I'm going to click step over again to get to line 8, then I'm going to click step into to go into my iterator call (in this case the anonymous function where I'm alerting).  Here's what I see:

<img src="{{ root_url }}/images/devtools_post/debug3.png" />

<br>

<img src="{{ root_url }}/images/devtools_post/debug4.png" />

For a problem this simple it may have seemed like overkill to use this debugging procedure in Dev Tools (especially when you can hit line 8 with a rock from line 19).
When your program gets big enough, though, that your functions call functions located in other files, or your recursive function has a bug somewhere in its 15th call to itself, Dev Tools will save your butt.

Practice this pattern!

As always, shoot any questions or conversations to <georgebonnr@gmail.com>