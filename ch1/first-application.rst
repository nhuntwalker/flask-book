=============
Hello, World!
=============

**Objectives**

- Download and install Flask in a virtual environment
- Create a simple webserver that prints "Hello, World!" to the browser
- Serve an HTML page from Flask

We want to get right to it and make sure we understand how Flask can work, so that we can build more interesting things sooner rather than later.
A good first step is to make Flask say "Hi".

Let's create a directory for this first step named ``hello-world``, a virtual environment within that directory named ``ENV``, then activate that virtual environment.

.. code-block:: shell

    $ mkdir hello-world
    $ cd hello-world
    $ python3.7 -m venv ENV
    $ source ENV/bin/activate

With our environment activated, we can download and install Flask from the `Python Package Index <https://pypi.org/>`_ using ``pip``.
For reproducibility I'll include the version number of every package I install with ``pip`` (and for that matter with ``npm``).

.. code-block:: shell

    (ENV) $ pip install Flask==1.0.2

While we're at it, we should keep track of everything that we install.
The conventional way to do this is with a ``requirements.txt`` file.
While we could use ``pip freeze`` to print out and automatically populate our ``requirements.txt``, we have to remember that ``pip freeze`` will list *every* package installed within your environment, including the dependencies.
This can sometimes cause issues when a package has system-dependent dependencies.
As such, we'll only include the packages in ``requirements.txt`` that we know we'll need.

.. code-block:: shell

    # within requirements.txt
    Flask==1.0.2

Now we can actually start writing our "Hello, World!" application!
Create an ``app.py`` file in your current directory.

.. code-block:: shell

    $ touch app.py

Open ``app.py`` in your favorite editor/development environment and fill it with the following:

.. code-block:: python

    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def home() -> str:
        """The Home route.
        
        This endpoint serves as the home route for our Flask application.
        It returns the "Hello World!" string no matter what.
        
        Returns
        -------
        str
            The hardcoded string "Hello, World!"
        """
        return "Hello, World!"

At the top, we're importing the ``Flask`` object from the ``flask`` package.
This is standard for importing an object from an external package.

We then construct our application, called ``app``, as an instance of the ``Flask`` object given the ``__name__`` attribute of our file.
The ``Flask`` object will use the filename as a reference for loading other ``.py`` files.
It won't be important for this project, but will come into play later on.

By the way, our application instance can be called anything.
However, convention is for it to be called ``app``, and maintaining that convention will make it easier for us to reference documentation for other packages that leverage Flask.

The next line down declares our application's first route as ``"/"``.
**Whatever is within that string will be appended to our application's hostname verbatim**, so we have to make sure that we include that leading slash on any route that we declare.

``app.route("/")`` is a decorator, which binds the route that we're declaring within the string to the function that follows.
Effectively we're saying: "Whenever a user accesses the route ``/``, this function should run."
This decorator can do a little more than just declare a route, and we'll see that later on.

Finally we're getting to our function declaration.

.. code-block:: python

    def home() -> str:

Moving forward, I'll be referring to any function that's directly bound to an application route as an **endpoint function**.
Endpoint functions can be named pretty much anything, but we should take care to name them according to what they do.

This endpoint function serves the home route, therefore I've called it ``home``.
It takes no parameters, so its parentheses are empty.
I've included a type hint[#f2]_ for the return value, showing that this function should be expected to return a string of some kind.

Next is a documentation string (hereon "doc string") that's far longer than the function it describes.
This is good within reason; we want to clearly describe what the function intends to do, what parameters it takes (if any), and what it'll be returning (if anything).

Finally we have the return value.

.. code-block:: python

    return "Hello, World!"

In this instance, the return value is the string ``"Hello, World!"``
Flask requires that its endpoint functions return either a JSON serializable object, like a ``list``, or ``dict``; an object that can be rendered directly within the browser, like a ``string``; or a ``Response`` object that itself contains one of the previous two options.
Because this is our first step, we'll only be rendering a string.