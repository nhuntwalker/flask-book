================================================
The To Do List - Flask API and a React JS Client
================================================

Now that we've dipped our toes in a bit with Flask, let's jump in and build something more substantial.
We'll continue to build increasingly-involved projects as we move along through this book, but we can start moving toward a full-stack web application now with a To Do list.

Why a To Do list?
While this sort of application is still fairly simple, it illustrates well the basic operations of a modern web application:

- interacting with a database
- reading data from that database and displaying it
- adding new data to the platform
- editing existing data
- deleting existing data
- working with a functionally-separate client written with a modern front-end framework
- deployment to a modern cloud platform

If at any point you'd like to see the finished product, along with a branch-by-branch walkthrough of the codebase, you can `visit the github repo for this chapter <https://github.com/Flask-Web-Development-Projects/chapter-01>`_.
Note that what's there will contain most of the same code but not all, especially as we approach deployment.

Getting Started from the Ground Up
----------------------------------

Let's start by creating a new directory for our To Do list, separate from what we were working on before.
If you have a directory on your system where you put projects, feel free to put this there.
You could also choose to just make it in your home directory.

We'll initialize a new git repository within that directory, as well as a new virtual environment.
We'll make sure to add our ``ENV`` directory to a ``.gitignore`` file to make sure that our environment (and whatever secrets we'll hide within) doesn't get committed.

.. code-block:: shell

    $ mkdir todo-list-server
    $ cd todo-list-server
    $ git init
    $ python3.7 -m venv ENV
    $ source ENV/bin/activate
    (ENV) $ echo 'ENV' > .gitignore

The First Route
---------------

Before we start writing any code toward our project, let's start a new working branch and switch to that branch.
We'll create an ``app.py`` file while on this branch, and commit.

.. code-block:: 

    (ENV) $ git checkout -b first-route
    (ENV) [first-route] $ touch app.py
    (ENV) [first-route] $ git add app.py .gitignore
    (ENV) [first-route] $ git commit -m 'First commit of the To Do list application'

To start, we're going to want two packages to help us create a Flask server that can interact with a client without much explicit configuration:

- `Flask API <https://www.flaskapi.org/>`_: a drop-in replacement for Flask that lets you visualize your API better, wraps your responses in some base-level headers (like ``Content-Type: application/json``), and provides built-in status codes for use in your endpoint functions' responses.
- `Flask CORS <https://flask-cors.readthedocs.io/en/latest/>`_: wraps your Flask app and provides the base-level CORS headers that allow your server to talk to external clients without you having to configure CORS yourself

Let's ``pip install`` these two packages, and save the versions we use in a ``requirements.txt`` file.
This file will become important later, when we deploy.

.. code-block:: shell

    (ENV) [first-route] $ pip install flask-api==1.1 flask-cors==3.0.7
    (ENV) [first-route] $ echo 'flask-api==1.1' >> requirements.txt
    (ENV) [first-route] $ echo 'flask-cors==3.0.7' >> requirements.txt
    (ENV) [first-route] $ git add requirements.txt
    (ENV) [first-route] $ git commit -m 'Created the requirements.txt file and added flask-api and flask-cors'

With those installed, we can start to fill in ``app.py``.
The first endpoint we want to write will serve a list of whatever task items we want to see on our client.
This endpoint will take no parameters, and will do nothing special (for now) besides respond with a list of tasks.
Since we don't yet have a database filled with items, we're going to have to fake it a bit.
Here's some fake data to start:

.. code-block:: python

    EXAMPLE_TASKS = [
        {"_id": "zV9tzagZSvq1Dw", "body": "Go grocery shopping", "complete": False, "creation_date": '03 April 2019 08:33:04 UTC'},
        {"_id": "kt3GjBprgC70Jw", "body": "Go to BodyPump on Friday", "complete": False, "creation_date": '08 April 2019 12:41:21 UTC'},
        {"_id": "Z1uDnlgOPAborQ", "body": "Clean the bathroom", "complete": False, "creation_date": '11 April 2019 16:03:35 UTC'},
        {"_id": "iQzuWBVkxdBV1g", "body": "Change the Brita filter", "complete": False, "creation_date": '15 April 2019 13:22:11 UTC'},
        {"_id": "rEHK2icXi8oWVQ", "body": "Meet Marcus for lunch", "complete": False, "creation_date": '27 April 2019 04:27:59 UTC'},
    ]

We won't need this later, but for now I'm going to put these fake tasks into a file called ``tasks.py`` that I can import from.

With that, we can populate ``app.py``.
Starting with ``Flask-API`` will look the same as when we started with ``Flask`` before.
We'll also want to import ``Flask-CORS``, and use it to enable cross-origin resource sharing on all routes, for all origins and HTTP methods.

.. code-block:: python

    # in app.py
    from flask_api import FlaskAPI
    from flask_cors import CORS

    from tasks import EXAMPLE_TASKS 

    app = FlaskAPI(__name__)
    CORS(app)

    @app.route("/api/v1/tasks", methods=["GET"])
    def get_tasks() -> list:
        """The Get Tasks route.

        This endpoint serves a list of task items that'll be consumed by 
        the client.

        Returns
        -------
        list
            A list of incomplete tasks.
        """
        return EXAMPLE_TASKS

Seeing is believing, so let's set our ``FLASK_APP`` and ``FLASK_ENV`` environment variables in ``ENV/bin/activate`` and fire up our Flask server.

.. code-block:: shell

    # in ENV/bin/activate
    export FLASK_APP="$(pwd)/../app.py"
    export FLASK_ENV=development

    # in the shell
    (ENV) [first-route] $ source ENV/bin/activate
    (ENV) [first-route] $ flask run

Now, in our browser, we can go to ``http://localhost:5000/api/v1/tasks`` and see our handiwork on display.

Note how we're not going to the base-level route ``http://localhost:5000/``.
That route doesn't exist for this project.
The only route we've declared thus far, and therefore the only route that exists for our application, is ``http://localhost:5000/api/v1/tasks``.

When we pop this open in the browser, we'll see some nice styling, the name of our endpoint function, whatever doc string we included on the function, the request that generated this response, and the data that the route would be passing back in JSON format (JavaScript Object Notation).

It worked!
Let's commit our code before we move forward.

.. code-block:: shell

    (ENV) [first-route] $ git add app.py tasks.py
    (ENV) [first-route] $ git commit -m 'Completed the first route with some sample data. Will remove sample data once the database is created.'

So why did we do this?
Why aren't we serving HTML, styling our response, and plugging in some JavaScript to make a pretty website?

In modern web development, in most cases, Flask will not be used to generate the full application from back to front.
As much as it can be used to put together a front-end with all the CSS and JS you might expect from a web application, the reality is that there are far better options for managing front-ends than Jinja, and Flask is better when optimized for its role as a server for an API (Application Programming Interface).
We can use Flask to retrieve, shape, and modify the data that our front-end needs to consume.
And if we need to scale it differently than our front-end, it'll be decoupled enough to allow that to happen.
Separating our server concerns from our client concerns allows us to keep each side focused on what it does best, instead of having a massive codebase that tries to handle all things.

Building the First View with ReactJS
------------------------------------

Of the many front-end libraries that can handle the job better than Flask+Jinja, one of the most popular as of this writing is ReactJS.
We'll use it extensively throughout this book.

The easiest way to get started with ReactJS is to use the ``create-react-app`` package via ``npx``.
We can do that with ``npx create-react-app <app name>``.
As mentioned above, we want to create a client that is wholly separate from our server aside from API calls.
Let's create this client in a new directory called ``todo-list-client``, separate from our server code.
Note: I'm going to be `creating this React app with TypeScript <https://facebook.github.io/create-react-app/docs/adding-typescript>`_, as adhering to strict typing helps when creating apps of any size.

.. code-block:: shell

    $ npx create-react-app todo-list-client --typescript

Navigating into this directory shows the foundation for a fully-functioning ReactJS application.
Note, as of this writing, I'm using React and React-DOM versions 16.8.6.
My directory looks like:

.. code-block:: shell

    $ ls
    .git          node_modules  src
    .gitignore    package.json  tsconfig.json
    README.md     public        yarn.lock

Running ``npm start`` will run the development server, serving code for the default React app and opening it in our default browser at port 3000.
``ctrl + C`` to kill the development server.

If you've never developed with React before, it may be difficult to tell where to go from here.
Fortunately, I have an idea.
First, let's understand what we're working with.

At the top-level we have:

- our git repo and ``.gitignore``
- ``README.md``
- ``node_modules``: houses all of the packages we either have installed or will install, including package dependencies
- ``package.json``: effectively the root of our app, listing the app name, version, author(s), dependencies, available scripts, and more. With this in our directory, we could wipe out our ``node_modules`` and rebuild them with an ``npm install``
- ``public``: contains the base ``index.html`` file that our React app hooks into, as well as the ``manifest.json`` which provides information about your application for mobile devices if a mobile user wants to install a shortcut to your web app on their phone. Generally, we'll only touch this directory if we need to change something within the ``<head>`` tag of the ``index.html`` file.
- ``src``: the meat of the application. Where all the JS and CSS is housed
- ``tsconfig.json``: the indicator that this codebase is written in TypeScript. This file dictates what is required to compile this codebase to JavaScript.
- ``yarn.lock``: a file storing the exact versions of packages that have been installed. If we were using ``yarn`` instead of ``npm`` to manage our packages, this file would update automatically as we installed more packages. However, we're using ``npm``, so a similar ``package-lock.json`` will be used.

Within the ``src`` directory, we have 

.. code-block:: shell

    $ ls src
    App.css            index.css          react-app-env.d.ts
    App.test.tsx       index.tsx          serviceWorker.ts
    App.tsx            logo.svg

Because our project is being written with TypeScript, all the files that would be ``.js`` files are now ``.ts`` or ``.tsx``.
Worry not, they all become JavaScript in the end.

``index.tsx`` is the actual root of our application, dictating how our React app hooks into the ``public/index.html`` file.
It'll import our whole app from ``App.tsx``, select the html element with the id of ``"root"``, and render our app within that element.
It'll also apply whatever styling is present in ``index.css``.

We won't be working with ``serviceWorker.ts``, so I'm going to pass over that for now.
We also won't be touching ``react-app-env.d.ts``, though it's worth noting that this is where the object types in ``react-scripts`` are being referenced.

``App.tsx`` is our application in its entirety, full-stop.
Anything we want on our front-end will go through here.
This file exports a React component called ``App``, which again gets imported into ``index.tsx``.
All React components that we work with or create ourselves will be capitalized to differentiate them from purely functional code.

Opening ``App.tsx`` reveals the following:

.. code-block:: javascript

    import React from 'react';
    import logo from './logo.svg';
    import './App.css';

    const App: React.FC = () => {
        return (
            <div className="App">
            <header className="App-header">
                <img src={logo} className="App-logo" alt="logo" />
                <p>
                    Edit <code>src/App.tsx</code> and save to reload.
                </p>
                <a
                    className="App-link"
                    href="https://reactjs.org"
                    target="_blank"
                    rel="noopener noreferrer"
                >
                    Learn React
                </a>
            </header>
            </div>
        );
    }

    export default App;

At the top of the file we have ``import React from 'react';`` which...imports React from the ``react`` package.
Any file that contains a component written with the JSX format we see in this file (where our DOM elements are written as raw HTML within a JavaScript file) will *need* to have this line at the top.

Next we have the importing of the ``logo`` image, which we can ignore, and the importing of the CSS styling from ``App.css``.

After all the imports we have the actual ``App`` component.
Note that it appears as a function, in this case an ES6 arrow function.
**Every React component effectively resolves to a function call that returns some code to be rendered as HTML.**