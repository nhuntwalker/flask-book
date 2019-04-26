====================================
Flask App with HTML and Static Files
====================================

As a developer, you may occasionally be asked to throw up a small website that serves some HTML.
As it currently stands, you could make that happen by providing raw HTML as a string in the ``return`` statement of your endpoint function like so:

.. code-block:: python

    def home() -> str:
        """...doc string..."""
        return """
    <html>
        <body>
            <h1>Hello, World!</h1>
        </body>
    </html>"""

The browser will take the HTML that you've returned and render it appropriately, but as you might imagine there's a variety of problems with this.
One is that when your HTML is substantial, this becomes ridiculously difficult to edit.
Another is that if you want to include any static files, you'll have to hardcode them and their relative paths, if you know what those paths might be, in this string.
Yet another is that if you have any dynamic data that needs to populate this HTML, you now have to do a bunch of string manipulation and concatenation to get it formatted properly to get it into your code.

Sure, these are all solvable problems, but in the words of `Raymond Hettinger <https://twitter.com/raymondh>`_, "There must be a better way!"

When you installed Flask, you also installed a templating library called `Jinja <http://jinja.pocoo.org/>`_.
With this, you can start with an HTML template in a separate file.
Then, you can use your application's data to programmatically generate content and properly point to static files that will be requested by your app.

Let's see this in action.

Templating with Jinja
---------------------

Let's take a step away from the Flask server and open up our Python prompt to see what we can create with Jinja.
It's already installed in your environment, so we don't have to ``pip install`` anything new.

.. code-block:: python

    Python 3.7.3 (default, Mar 27 2019, 09:23:39) 
    [Clang 10.0.0 (clang-1000.11.45.5)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> from jinja2 import Template

Jinja's ``Template`` will allow us to use some simple control language to populate a string with whatever data we want, somewhat like Python's own f-strings.
For example, let's say we want to dynamically fill an ``<h1>`` tag with a first and last name.

.. code-block:: python

    >>> tmpl = Template("<h1>Name: {{ first_name }} {{ last_name }}</h1>")

When we render the template, at that moment we provide the data we want to fill the template with.

.. code-block:: python

    >>> tmpl.render(first_name="DeSean", last_name="Rodgers")
    '<h1>Name: DeSean Rodgers</h1>'
    >>> tmpl.render(first_name="Kizzie", last_name="Dunlap")
    '<h1>Name: Kizzie Dunlap</h1>'

At this point we haven't done anything we couldn't do with an f-string.
Let's do a little more.

Let's say the page we're trying to render into HTML is an address book of Python core developers, stored like so:

.. code-block:: python

    >>> python_core_devs = [
    ...     {"name": "Guido van Rossum", "twitter_handle": "gvanrossum"},
    ...     {"name": "Raymond Hettinger", "twitter_handle": "raymondh"},
    ...     {"name": "Mariatta Wijaya", "twitter_handle": "mariatta"},
    ...     {"name": "Brett Cannon", "twitter_handle": "brettsky"},
    ...     {"name": "Emily Morehouse-Valcarcel", "twitter_handle": "emilyemorehouse"}
    ... ]

Jinja can make it more natural to, say, list these people within individual ``<div>`` elements while also including their Twitter handles with the appropriate links.

.. code-block:: python

    >>> tmpl = Template("""<section class="container">
    ...     {% for person in python_core_devs %}
    ...     <div>
    ...         <h2>{{ person.name }}</h2>
    ...         <p>
    ...             <a href="https://twitter.com/{{ person.twitter_handle }}">
    ...                 Twitter: @{{ person.twitter_handle }}
    ...             </a>
    ...         </p>
    ...     </div>
    ...     {% endfor %}
    ... </section>""")
    >>> print(tmpl.render(python_core_devs=python_core_devs))
        <section class="container">
            
            <div>
                <h2>Guido van Rossum</h2>
                <p>
                    <a href="https://twitter.com/gvanrossum">
                        @gvanrossum
                    </a>
                </p>
            </div>
            
            <div>
                <h2>Raymond Hettinger</h2>
                <p>
                    <a href="https://twitter.com/raymondh">
                        @raymondh
                    </a>
                </p>
            </div>
            
            <div>
                <h2>Mariatta Wijaya</h2>
                <p>
                    <a href="https://twitter.com/mariatta">
                        @mariatta
                    </a>
                </p>
            </div>
            
            <div>
                <h2>Brett Cannon</h2>
                <p>
                    <a href="https://twitter.com/brettsky">
                        @brettsky
                    </a>
                </p>
            </div>
            
            <div>
                <h2>Emily Morehouse-Valcarcel</h2>
                <p>
                    <a href="https://twitter.com/emilyemorehouse">
                        @emilyemorehouse
                    </a>
                </p>
            </div>
            
        </section>

Could you still do this with a large f-string?
Sure.
However, after a point it becomes absurd to fill your Python code with large-scale string formatting.
You also want to be able to write your HTML in separate files that at least look like HTML, whose syntax will be appropriately highlighted by your editor of choice.
For that, at least for now, you can use Jinja templating alongside your Flask code.

Pulling Jinja Templates into Flask
----------------------------------

Let's say we wanted to serve up that address book in the previous section from our Flask application.
By default, Flask will look for templates to render in a directory called ``templates``, located at the same level as our application code.
We can do ourselves a favor and set up a directory structure that looks like so:

.. code-block:: shell

    ENV
    app.py
    templates/

We can then copy that template string we wrote before and paste it into an ``index.html`` file located within the ``templates`` directory

.. code-block:: html

    <!DOCTYPE html>
        <body>
            <h1>Some Python Core Developers</h1>
            <section class="container">
                {% for person in python_core_devs %}
                <div>
                    <h2>{{ person.name }}</h2>
                    <p>
                        <a href="https://twitter.com/{{ person.twitter_handle }}">
                            Twitter: @{{ person.twitter_handle }}
                        </a>
                    </p>
                </div>
                {% endfor %}
            </section>
        </body>
    </html>

Looks like HTML, but includes some Jinja control syntax.

Now that our template is safely tucked within an html file, we can use Flask's ``render_template`` function to provide it the appropriate data and render it in the browser.
``render_template`` works exactly like ``Template.render`` did in our Python prompt, filling our template with data and returning the populated string.

.. code-block:: python

    from flask import Flask, render_template

    app = Flask(__name__)

    python_core_devs = [
        {"name": "Guido van Rossum", "twitter_handle": "gvanrossum"},
        {"name": "Raymond Hettinger", "twitter_handle": "raymondh"},
        {"name": "Mariatta Wijaya", "twitter_handle": "mariatta"},
        {"name": "Brett Cannon", "twitter_handle": "brettsky"},
        {"name": "Emily Morehouse-Valcarcel", "twitter_handle": "emilyemorehouse"}
    ]

    @app.route("/")
    def home() -> str:
        """The Home route.
        
        This endpoint serves as the home route for our Flask application.
        It returns the rendered HTML form of the Python core developers.
        
        Returns
        -------
        str
            The return value of a render_template call to the index.html template
        """
        return render_template("index.html", python_core_devs=python_core_devs)

Take a moment to run the server with ``flask run`` and gaze upon your work with an appropriately-satisfied smile.

Incorporating Static Files (CSS, JS, etc)
-----------------------------------------

Even the smallest websites have some style, and ours has none but the default.
If we wanted to include external stylesheets, JavaScript files, and images, all we'd have to do is include the links to those resources in our HTML, as we normally would.

For local static files, Flask will allow us to incorporate whatever we wish as long as it knows where to find it.
By default, it'll look for static files in a directory called ``static`` at the same level as your application file (currently named ``app.py``).
Let's help ourselves by adding a ``static`` directory to our directory tree, and within that directory a ``style.css`` and an ``app.js``.

.. code-block:: shell

    ENV
    app.py
    templates/
        index.html
    static/
        style.css
        app.js

Within the ``style.css`` file include the following code:

.. code-block:: css

    * {
        box-sizing: border-box;
        font-family: Arial, Helvetica, sans-serif;
    }

    h1 {
        background-color: #2b5b84;
        width: 600px;
        text-align: center;
        padding: 20px;
        color: white;
        font-weight: 100;
        margin: 0 auto;
    }

    section {
        width: 600px;
        text-align: center;
        margin: 0 auto;
    }

    h2 {
        background-color: #ffdf76;
        padding: 20px 0;
        font-weight: 100;
        margin: 0;
    }

    p {
        background-color: rgb(29, 161, 242);
        margin: 0 auto 10px;
        width: 60%;
        padding: 20px;
    }

    p a {
        text-decoration: none;
        color: white;
    }

And the ``app.js`` file should contain the following:

.. code-block:: javascript

    let divs = document.getElementsByTagName('div');

    for (let i = 0; i < divs.length; i++) {
        let time = 250 * (i + 1);
        let theDiv = divs[i];

        setTimeout(() => {
            theDiv.style.display = "block";
            for (let j = 0; j < 1.0; j += 0.01) {
                setTimeout(() => {
                    theDiv.style.opacity = j;
                }, time + (j * 250));
            }
        }, time);
    }

A note: you really shouldn't try to roll your own "sequential fade-in" code, but there you have it in case you wanted to have an idea of how it could be done.

Finally, we need to modify our Jinja template.
Note, we're not actually modifying the Python code at all.
The Flask server doesn't care about what the front-end does as long as the front-end isn't trying to make a call to the server.

In the Jinja template we're going to add the following to the top, above the ``<body>`` tag but below the ``<html>`` tag:

.. code-block:: html

    <head>
        <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
        <title>Python Core Devs</title>
    </head>

and then after the closing ``</body>`` tag we'll add:

.. code-block:: html

    <script src="{{ url_for('static', filename='app.js') }}" type="text/javascript"></script>

With these additions, we're loading our ``style.css`` file before the HTML body loads, and loading our ``app.js`` file after the contents of the body tag have finished loading.

We dynamically locate our static files using the ``url_for`` function call provided by Flask to the Jinja template.
``url_for`` will resolve the endpoint asked of it (the first argument to the function call), and in the case of trying to provide a file will join the ``filename`` argument with that endpoint, separating them with a forward slash ``"/"``.
The URLs to the static files are loaded in as the template renders, and by the time they hit the front-end they appear as ``/static/style.css`` and ``/static/app.js``.

And thus we have a webpage!
Feel free to refresh the page a few times to witness the glory of divs fading in over and over again.
I know I did.