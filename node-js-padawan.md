
<!--
name: node-js-padawan
freshnessDate: 2014-12-12
version : "0.1.2"
title : "7 Tips for a Node.js Padawan"
description: "What I wish I knew when I started."
homepage : "https://medium.com/@faisalabid/7-tips-for-a-node-js-padawan-e7c0b0e5ce3c"
author : "Faisal Abid"
license : "Unknown"
contact : { url: "http://linkedin.com/in/faisalabid" }
-->

<!-- @section -->

# Introduction

Node.js development is extremely fun and satisfying. There are over 35k modules to choose from, and overall node is very easy to develop a working application that can scale easily.

However for developers just starting off with Node.js development, there are a few bumps along the road. In this short post I cover a few of the things I questioned and ran into while learning Node.js.

<!-- @section -->

# Tip 1: Use nodemon for development. pm2 for production.

When you first get started with Node.js development, one of the things that will stick out like a sore thumb is having to run node [file].js over and over again. When I got started with node, this was extremely frustrating and painful. Especially having to control C every time I modified something.

Luckily I discovered a great tool called [Nodemon](https://github.com/remy/nodemon).

<!-- @REMOVETHISlink, "url" : "https://github.com/remy/nodemon", "task" : "Install Nodemon by running npm install -g nodemon." -->

Nodemon is an awesome tool, once you install it globally you can run your node.js scripts via nodemon [file].js. Doing so will tell nodemon to monitor your script and all the scripts that it depends on for changes. This is an awesome way to do Node.js development and speeds everything up.

What about production? Unless you are using Heroku, Nodejitsu or other great Node.js hosting providers, chances are you will be using EC2 or another cloud provider to run your Node.js app. How do you properly run a Node.js app to make sure it’s always running?

The answer to that question is a great tool called [PM2](https://github.com/Unitech/pm2).

<!-- @REMOVETHISlink, "url" : "https://github.com/Unitech/pm2", "task" : "Install PM2." -->

PM2 is a tool like nodemon which is intended to run your node app in production. Like Nodemon it will monitor your app for changes and redeploy them, but unlike Nodemon, if PM2 encounters a crash, it will restart your node.js app right away.

Where PM2 excels though is when you need to scale your app to multiple cores. PM2 comes with a built in “load balancer” that lets you easily specify how many instances of your Node app to run.

```javascript
pm2 start app.js -i max
```

The -i parameters lets you specify how many instances to run, in this case PM2 comes with a built in constant called max which auto scales your app to the amount of cores you have. Remember Node runs only on one core!

<!-- @section -->

# Tip 2: Async or Q

The more you start to write node.js apps, the sooner you’ll realize the pain of callback hell. If you don’t know what callback hell is, here is an example:

```javascript
 function register(name, password, cb){
  checkIfNameExists(name, function(err, result){
   if(err){
    return cb(“error”);
   }
   checkIfPasswordGood(password, function(err, result){
    if(err){
     return cb(“error”);
    }

    createAccount(name,password, function(err,result){
     if(err){
      return cb(“error”);
     }
     createBlog(name, function(err, result){
      sendEmail(name, function(err, result){
       callback(result);
      });
     });
    });
   });
  });
 }
```

While not a very useful or amazing block of code, it should get the point across that callback hell is a very real thing. But how do you avoid this?

One simple way is to use events. I personally don’t like using events because then you are using events to call private functions which have only one purpose, which defeats the point of a function.

How do you do this then? There are two competing libraries out there, async.js and Q. Both offer their own take on how callback hell should be prevents.

[Async.js](https://github.com/caolan/async) or “async” allows you to easily execute functions in series or parallel without the need of nesting them back to back.

<!-- @REMOVETHISlink, "url" : "https://github.com/caolan/async", "task" : "Install Async.js." -->

Below are some of the patterns that Async supports taken from their readme. For a list of all the patterns async supports check out their repo.

```javascript
 async.map([‘file1',’file2',’file3'], fs.stat, function(err, results){
  // results is now an array of stats for each file
 });

 async.filter([‘file1',’file2',’file3'], fs.exists, function(results){
 // results now equals an array of the existing files
});

 async.parallel([
  function(){ … },
  function(){ … }
  ], callback);

 async.series([
  function(){ … },
  function(){ … }
  ]);

 async.waterfall([
  function(callback){
   callback(null, ‘one’, ‘two’);
  },
  function(arg1, arg2, callback){
   callback(null, ‘three’);
  },
  function(arg1, callback){
 // arg1 now equals ‘three’
 callback(null, ‘done’);
 }
 ], function (err, result) {
 // result now equals ‘done’
});
```

If we take what we did previously with register, we can apply the waterfall pattern in async. The result of this is a very readable code pattern that doesn’t involve the pyramid of doom.

[Another great library is Q](https://github.com/kriskowal/q). This library is exposes the concept of promises. A promise is basically an object that is returned from a method with the “promise” that it will eventually provide a return value. This ties is very neatly with the asynchronous nature of javascript and node.js.

<!-- @REMOVETHISlink, "url" : "https://github.com/kriskowal/q", "task" : "Install Q." -->

For example, taken from Q’s repo page.

```javascript
 promiseMeSomething()
 .then(function (value) {
 }, function (reason) {
 });
```

The promise me function returns an object right away. Calling then on the object will call the function you pass in with the value you want returned. Then also takes an additional callback which is run when the object fails to return the value.

This is a very neat way to avoid the insanity of callback hell. If we take our registration example, you can easily make it so that each of those functions is called when then is executed.

```javascript
 Q.fcall(checkIfNameExists)
 .then(checkIfPasswordIsGood)
 .then(createAccount)
 .then(createBlog)
 .then(function (result) {
 // Do something with the result
})
 .catch(function (error) {
 // Handle any error from all above steps
})
 .done();
```

As I said previously, I dislike creating single purpose functions. Instead of passing in the function name to “then”, I would just create an anonymous inner function and pass that in, however the choice is yours.

In summary, if you start to realize you’re creating callback hell for yourself then it’s time to look into async.js or Q.

My personal fav? **Q all the way**!

<!-- @section -->

# Tip 3: Debugging Node.js apps easily

Debugging Node.js apps will be confusing if you are coming from a language with heavy IDE integration like Java or C#. Most new node developers adopt the “flow” debugging pattern, where your best friend becomes console.log.

However there are still alternatives that are more convenient for debugging. Node.js comes with a built in debugger that you can run by calling node debug, however the one I love is node-inspector.

Taken from their github repo “Node Inspector is a debugger interface for node.js using the Blink Developer Tools (former WebKit Web Inspector).”

In a nutshell node-inspector lets you debug your applications using whatever editor of your choice and chrome web tools. That is sexy.

Node-inspector lets you do some really cool things like live code changing, step debugging, scope injection and a bunch of other cool stuff.

It’s bit involved to setup, so I’ll let you [follow the instructions yourself](https://github.com/node-inspector/node-inspector).

<!-- @REMOVETHISlink, "url" : "https://github.com/node-inspector/node-inspector", "task" : "Install node-inspector." -->



<!-- @section -->

# Tip 4: Nodefly

Once you have your application up and running, you might ask yourself how you could monitor it’s performance and profile it to make sure your app is running at optimum speed. The simplest answer to that is a great service I use called Nodefly.

Nodefly with a simple line of code starts to monitor your application for memory leaks, measure how long it takes for redis, mongo queries and a bunch of other cool stuff.

<!-- @link, "url" : "https://strongloop.com/strongblog/strongloop-acquires-the-assets-of-nodefly/", "task" : "Read about Nodefly." -->


<!-- @section -->

# Tip 5: Module management with NPM.

One of the most common things to do in node is installing packages via NPM. Node has an amazing package manager which installs all the modules specified in your package.json manifest file. However one thing all beginers run into is keeping this package.json file updated with all the modules you are using.

It seems like a pain to always be opening your package.json to update the dependcies property with the new module you just installed, but what many don’t know is npm will do this for you!

Simple run `npm install --save module_name` and npm will automatically update your package.json with the correct module and version name.

<!-- @section -->

# Tip 6: Don’t check in your node_modules folder

While we are on the topic of modules and npm, not many know that you shouldn’t check in your `node_modules` folder. The biggest reason behind this is there is no need for you to check this folder in. Whenever someone checks your source out they can just run npm install and download all the modules required.

You might say that it’s not a big deal if you check in `node_modules`, but what if the person checking out your source is using an operating system other than yours and one of the modules that your app uses is compiled when it’s installed via npm? Your app will crash and the person that checked out your source will have no idea why!

For example modules like bcrypt and sentimental are compiled on the host system when you install them because they have native components written in C.

The best way to avoid checking in your `node_modules` folder is by adding it to .gitignore.

// .gitignore node_modules/*

<!-- @section -->

# Tip 7: Don’t forget to return

A common mistake made by all early node developers is forgetting to return after a callback. While some times this has no implications, there are many times where you’ll run into odd issues because your callback is being called twice.

Lets take a look at a quick example

```javascript
 function do(err,result, callback){
 if(err){
 callback(“error”);
 }
 callback(“good”);
 }
```

At first glance, this snippet makes sense. If there is an error, send “error” in the callback, if not send good. But calling the callaback is not stopping the method from completing the execution. It will just move on to calling callback(“good”).

Inside long and complex lines of code, doing this will save you hours and hours of debugging.

Node.js is great platform to develop on. If you keep these 7 things in mind while developing, debugging and deploying to production you can save your time and prevent your hair from graying out.
