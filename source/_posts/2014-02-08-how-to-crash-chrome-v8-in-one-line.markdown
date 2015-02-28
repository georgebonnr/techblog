---
layout: post
title: "How to Crash Chrome V8 in One Line"
date: 2014-02-08 14:29
comments: true
categories: javascript
keywords: javascript, v8
description: 
---


One of the best pranks you can play on JavaScript is setting a length property on an object and giving it a negative value.

Example:
    var foo = {
      length: -5
    };

I decided to experiment the other day with what kind of trouble I could cause with this.
What exactly a negative length is, and what would have to go horribly wrong for you to encounter this scenario in a real project is a subject for another (longer) post.

Something that can produce unexpected and fun results is calling array methods on objects.  For instance:

    var obj = {foo: "bar", baz: "qux"};
    Array.prototype.push.call(obj, 'thud');

returns
`1`.

And obj is now

`Object {0: "thud", foo: "bar", baz: "qux", length: 1}`

Similarly we can call

`Array.prototype.pop.call(obj)`

Which returns `"thud"` and leaves obj as `Object {foo: "bar", baz: "qux", length: 0}`

Huhm.

So, back to our object, `var foo = { length: -5 };`

One of the first things I tried was `Array.prototype.slice.call(foo)`.

Screeeeeeeeeech.  (this is where you might see a gif of something exploding or a small animal falling over if I weren't already paranoid about rampant gif-pandering on programming blogs).

Pro tip: if your browser process is crashed, click the hamburger icon, select tools, and go to Task Manager.  Chrome will let you kill the individual tab with the crashed process.

At this point I was pretty curious about what what made this fail so catastrophically.

I looked up the V8 message board, and lo and behold, somebody had recently submitted an issue about this same thing, but nobody had investigated it yet.

So I dove into the source code – you can either build it locally, or check out [the official mirror on github](https://github.com/v8/v8).

After some digging, I traced the issue to line 684 in the slice method in [array.js](https://github.com/v8/v8/blob/d916860bc096b8824dbb4459c1b9c5cff2fca182/src/array.js) (V8 binds its own method called ArraySlice, which delegates to two different helper methods depending on the size of the array). 

The `TO_UINT32` constant turned out to be a Python macro [stored here](https://github.com/v8/v8/blob/e9a2eb091b2801cb98333edb44ba7a68611fe886/src/macros.py).

The macro uses a bitwise unsigned right shift operator to quickly convert the argument to a positive integer, if it wasn't already.  However, when a negative value is shifted in this manner, unpredictable things can happen – usually an extremely large positive number is returned. In this case `-5 >>> 0` will return `4294967291`.

So the length of the array or object is set internally to `4294967291`, which in turn sets an end index for handling the particulars of the slicing, and it's all downhill from there.

Some of the array methods don't break in this same manner – namely, the ones that don't rely on the length of the array being accurate to succeed.  However, quite a few of them do crash when passed an object with a negative length property.  For instance, `Array.prototype.shift.call({length: -5})`.

While this is, again, something that would hardly ever come up in production, I posted about the issue, and it led to a feature request to change this implementation in ES6 / Harmony.
Yay, the internet!

I'm sure there are tons of other 'one-liners' out there that can kill your browser – more than just the usual suspects like  `while (true) {true}`.  Anybody got any others they want to share?
