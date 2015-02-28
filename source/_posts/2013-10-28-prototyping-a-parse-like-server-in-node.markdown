---
layout: post
title: "Prototyping a Parse-like Server in Node"
date: 2013-10-28 03:12
comments: true
categories: javascript, node
keywords: javascript, node, parse, server
description: Writing an HTTP server from scratch in Node
---

Parse is a REST-ful backend-as-a-service that’s easy to set up and use with your app.  I highly recommend it. Today, however, I’m going to show you how to take a standard chat application and go from using Parse to using our own server written in Node. Why? Because it’ll be good for you (you’ve been meaning to learn Node for a while now, right?). 

We’re also going to implement a Parse-like feature (sorting) in our server. I’m going to show you most of the key points along the way, but if you're coding along at home the implementation of some of these steps will be up to you (you can refer to my example code [here](https://github.com/georgebonnr/NodeBackend ) if interested). Let’s get started!


The client side of our chat application uses AJAX calls in jQuery to communicate with Parse (if AJAX calls are new to you, check out [this resource](http://www.jquery4u.com/ajax/key-differences-post/)). Here’s what it looks like:
    var getData =  function() {
     $.ajax({
       url: 'https://api.parse.com/1/classes/chatterbox',
       type: 'GET',
       data: {
         order: "-createdAt"
       },
       success: function (data) {
         render(data.results);
       },
       error: function (data) {
         console.error('Failed to retrieve messages');
       }
     });
    };

The success and error properties define callback functions that will run once the response comes back from the server (hopefully containing our chat data).

We’re going to run our Node server on our local computer for now, so the main thing that’s going to change in our AJAX call is simply the url we are sending the request to. We're also going to add the ability on our server to filter messages into different rooms and only return messages for the client's current room when they request data.
    var getData =  function() {
     $.ajax({
       url: 'http://127.0.0.1:8080/classes/chatterbox',
       dataType: 'json',
       type: 'GET',
       data: {
         order: "-createdAt",
         room: currentRoom // this is a property set in the client-side code.
       },
       success: function (data) {
         console.log(data);
         render(data);
       },
       error: function (data) {
         console.error('chatterbox: Failed to retrieve messages');
       }
     });
    };
Let's set up the bare bones of our Node server.  Node includes a module that makes it easy to handle http requests, the rather understatedly-named `http`.  Here's what our basic server will look like:

    var http = require("http");
    var handler = require("./request-handler");
    var port = 8080;
    var ip = "127.0.0.1";
    var server = http.createServer(handler);
    server.listen(port, ip); 

As you can see we're defining our process for handling requests in a separate file to keep things cleaner, since that's where most of the logic in our server will go.  The `require` statements are how Node loads modules, whether they are built in modules like `http` or separate files that we happen to write.  In `request-handler` we'll set any variables or functions that we want to make available via the `require` statement as properties on a `module.exports` object (we can even define `module.exports` as a single function, which is what we happen to be doing here). 

One of the appealing things about Node is its baked-in emphasis on privacy – it makes you explicitly state which parts of your code you want to make available to other files rather than loading the entire file by default (as we would do in a script tag in plain javascript).

Our request handler will need to know two things to start with: what route the client is requesting (as well as what parameters, if any, are serialized into the url after the route), and what type of request has been sent (here we are just dealing with GET and POST requests).

One way to set up a router for very simple applications like this is to define an object with the requested routes as keys and the appropriate handler actions to take as the corresponding values.  Again, since this is Node we can write pretty much anything we want in plain old Javascript. Here's what a simple router might look like on our back end:

    var router = {
      '/messages': storageAccess,
      '/getrooms': getRooms,
      '/': serveFile,
      '/styles/styles.css': serveFile,
      '/scripts/app.js': serveFile,
      '/scripts/config.js': serveFile
    };

We'll need to send a response back to the client with a status code, headers, response body, and a content-type property in the headers.

We can set the default response to a status code of 404 (not found) with an appropriate message in the response body (the message will be received by the client and passed into the success callback).

To get the route that the user is requesting out of the url we can require the (again appropriately named) `url` module in Node and use the `parse` method to get a string that we can work with.  Then we can check to see if that route exists in our object and if so call the appropriate function in our file which will redefine the status code, headers, and response body as needed.  If we have not defined the route as one we can handle, then we'll simply pass along the 404 default response. Here's what that process looks like:

    var pathname = url.parse(request.url).pathname;
    if (router[pathname]) {router[pathname]();}
    response.writeHead(statusCode, headers);
    response.end(responseBody); // response.end will send along whatever message is passed into it in addition to closing the transfer

If you'll notice the last two lines will be executed with every request. For this example we're not doing any asynchronous operations in Node so this setup works fine.  However if we were reading from files asynchronously or reading from a database we would want to call response.write() and response.end() in the appropriate callbacks to those actions.

Just as a reminder, if you'd like more guidance on the details of the implementation you can check out the finished code [here](https://github.com/georgebonnr/NodeBackend )

`storageAccess` will, for now, simply grab messages and a list of rooms from in-memory storage (via getter and setter functions).  `getRooms` will do something similar.  Here are example get and set functions – you can also see how we can handle simple ordering on our server (the `options` argument will be an object parsed from the url query.)

    var set = function(message){
      message.createdAt = new Date();
      messages[message.room] ? messages[message.room].push(message) : messages[message.room] = [message];
      fs.appendFile("log.txt", JSON.stringify(message));
    };
    var get = function(options){
      if (messages[options.room] === undefined) { messages[options.room] = []; }
      var roomMessages = messages[options.room] || [];
      if (options.order) {
        return roomMessages.sort(function(a,b) {
          if (options.order[0] === "-") { return b[options.order.slice(1)] - a[options.order.slice(1)]; }
          return a[options.order] - b[options.order];
        });
      }
      return roomMessages;
    };

The `fs.appendFile` line makes use of another built-in Node module.  `fs` stands for filesystem, and it allows most server tasks related to reading and writing files to be handled both synchronously and asynchronously.

The last thing we'll look at is how we use `fs` to serve up html, css, and client-side javascript pages on page load. The first step for our request handler is of course to require the fs module: `var fs = require('fs');` Next, if you'll recall we mentioned a serveFile function in our router for serving up our assets.  Here's what that function will look like:

    var serveFile = function(){
      statusCode = 200;
      if (pathname === '/') {
        headers['Content-Type'] = "text/html";
        responseBody = fs.readFileSync('../client/index.html');
      } else {
        headers['Content-Type'] = (pathname === '/styles/styles.css') ? "text/css" : "text/javascript";
        responseBody = fs.readFileSync(path.join('../client',pathname));
      }
    };

`fs.readFileSync` refers to the synchronous method for reading a file – it will "block" execution of all other code until it is done reading a given file and serializing its contents.  Since we are dealing with a relatively small html file this will work fine for our purposes, but much of the power of using Node on the back end lies in its built-in methods for reading and writing data asynchronously.

Check out the [fs documentation](http://nodejs.org/api/fs.html) and [this walkthrough](http://nathansjslessons.appspot.com/lesson?id=1085) for more info on asynchronous alternatives and try some of them out if you're feeling feisty (remember you'll need to wait until your file is completely read before you send back your http response, which means you'll need to do that work in a callback. To avoid repeating `response.writeHead` and `response.end` code in every one your routes, try defining a single `responseSet` function that each of your routes can call, passing in the appropriate headers and response body as arguments).

There are quite a few details that we didn't go over, but hopefully this gives you a broad overview of a server's basic functionality, and how you can implement that functionality in Node.  Check out the example backend code [here](https://github.com/georgebonnr/NodeBackend), and keep exploring the wonderful world of Node!