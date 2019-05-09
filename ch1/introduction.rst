===============================================
Introduction to Flask and Python Web Frameworks
===============================================

The world of Python web frameworks is vast, with frameworks existing for applications large and small.
There's frameworks like Django, which is made for large, sprawling applications and comes with many batteries included as well as many opinions about how applications should be built.
There's also frameworks like aiohttp, which provides an asynchronous http client and server, written for Python.
There's even serverless options, like Chalice, which looks a lot like Flask, but is built to allow you to deploy applications to AWS Lambda.
As with any toolset, you choose what makes sense for the problem you're trying to solve.

The Flask web framework [#f0]_ is about as simple as you can get with a web framework, offering very little on its own beyond the ability to receive HTTP requests and provide responses to those requests.
While the development community around the framework is lively, the framework itself is bare-bones.
It's meant for small web applications with limited scope.

But that doesn't mean it has to remain that way.

What I hope to show you throughout this book is a variety of ways in which the Flask web framework can be used to build applications of varying sizes, shapes, and scopes.
We'll see how the community around Flask has stepped in to pick up where the framework leaves off, allowing connectivity to a variety of tools that help make robust web applications.
We'll also see how we can link our server-side Flask applications to modern front-end libraries, providing rich user experiences while keeping our server focused on doing what it does best.

I'm happy that you're ready to take this journey with me, so without further ado let's dive right into it.
