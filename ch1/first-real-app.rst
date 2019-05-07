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
This endpoint will take no parameters, will only accept incoming "GET" requests, and will do nothing special (for now) besides respond with a list of tasks.

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

Here we've said that our route should match "<the host>/api/v1/tasks", and that the only HTTP methods that will be accepted for such a match will be "GET" requests.
If we wanted, we could allow for this route to handle more than one type of HTTP request. 
However, we're trying to build a REST-like API, where the incoming HTTP methods and the routes they access have meaning.
The route dictates which resources should be accessed, and the method lets us know how to access them.

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
Let's commit and merge our code before we move forward.

.. code-block:: shell

    (ENV) [first-route] $ git add app.py tasks.py
    (ENV) [first-route] $ git commit -m 'Completed the first route with some sample data. Will remove sample data once the database is created.'
    (ENV) [first-route] $ git checkout master
    (ENV) [master] $ git merge first-route -m 'Created the first server-side route for returning a list of task items'

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
Install the packages for this app all at once with ``npm install``.

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
Finally, we can remove ``App.test.tsx``, as we won't be writing tests for the duration of the book.

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

After all the imports we have the actual ``App`` component, which we can see is of type ``React.FC`` (short for "FunctionComponent").
Note that it appears as a function, in this case an ES6 arrow function.
This is one of the two ways we can write React components:

- a function that returns code that renders to HTML when called
- a class that inherits from ``React.Component`` and renders HTML when its ``render`` method is called.

Let's clear out everything within the ``<div className="App">`` element so we can start to build our first view.
Let's also change the type of App from ``React.FC`` to just ``FunctionComponent``, making sure to actually import ``FunctionComponent`` from ``'react'``.
Also, let's also remove the ``logo`` import since we won't be using it.

Our ``App.tsx`` file should now look like this:

.. code-block:: javascript

    import React, { FunctionComponent } from 'react';
    import './App.css';

    const App: FunctionComponent = () => {
        return (<div className="App"></div>);
    }

    export default App;

Note: while we don't necessarily need the parentheses around that ``<div>`` element, they become useful when what you're returning grows to be a lot and you want to move things onto separate lines.
React will try to insert a semicolon at the first opportunity when it sees ``return``.
Having the parentheses secures that you get to return what you're expecting.

Let's commit before continuing, as this is a nice starting point for our client application.

.. code-block:: shell

    $ git checkout -b list-tasks
    [list-tasks] $ git add package.json src
    [list-tasks] $ git commit -m 'Cleared out the App component and ready to fill with data.'

Let's discuss what we're about to do before we do it.
The first thing we're going to want this view to do is retrieve whatever task items are available from the server, and list those task items on the page.
How they're listed isn't necessarily important, just as long as they're on the page.
Let's use a list of divs.

Because we're expecting an array of task items, let's define in our code what a task item should look like.
We'll define it in a separate file called ``types.tsx``.
Let's create the file in the ``src`` directory and fill it with our type definition for a ``Task``.

.. code-block:: javascript

    // in src/types.tsx
    export interface Task {
        _id: string;
        body: string;
        complete: boolean;
        creationDate: string;
    };

With TypeScript, when we're defining what properties a given object should have we create an ``interface``.
An interface defines the property types one by one, ending each property type definition with a semicolon.

The properties on this object will mirror the "fake" data we'll be getting from our server.
As such, the "``_id``", "``body``", and "``creationDate``" fields will always be strings, and the "``complete``" field will always be a boolean.

For this project we're only going to have one type of object, but it's a good habit to get into to build our type definitions in a separate file and import them when and where we need them.

This is a significant change, so let's commit!

.. code-block:: shell

    [list-tasks] $ git add src/types.tsx
    [list-tasks] $ git commit -m 'Created the Task interface'

Now that we have our type definition, let's go back to "``src/App.tsx``".
We can import our ``Task`` type from ``types.tsx`` at the top.

.. code-block:: javascript

    import React, { FunctionComponent } from 'react';
    import { Task } from './types';
    import './App.css';

    const App: FunctionComponent = () => {
        return (<div className="App"></div>);
    }

    export default App;

To move forward from here we're going to have to introduce some state to this component.
For this, we can use React's ``useState`` function.

.. code-block:: javascript

    import React, { FunctionComponent, useState } from 'react';

``App`` will be getting these ``Task`` items from our server and using them to populate some divs.
Initially though, it won't have any Tasks to work with.
We can reflect that within our component with ``useState``.

.. code-block:: javascript

    // in src/App.tsx
    const App: FunctionComponent = () => {
        const [ tasks, setTasks ] = useState<Array<Task>>([]);
        return (
            <div className="App"></div>
        );
    }

We won't be using ``class``-based React components, so we won't have access to the state and state-based functions that are inherent to those types of components.
Instead we'll be using ``useState`` to affect the state of our components.
Later, we'll also use ``useEffect`` to decide when to re-render our component.

The ``useState`` function provided by React, also known as the `State Hook <https://reactjs.org/docs/hooks-state.html>`_, takes a value of any type that you wish to represent part or all of the initial state of your component.
Given this initial state value, it'll return two things for your use:

-  copy of that initial value
- a function that can update the state of the component

In the example above we wrote

.. code-block:: javascript
    
    // at the top
    import React, { FunctionComponent, useState } from 'react';

    // within the function
    const [ tasks, setTasks ] = useState<Array<Task>>([]);

Effectively what we said was "We expect our state for this function to at any time consist of an Array of Task items.
Initially, that array will be empty. 
Pass us that empty array to start, as well as a function to update what that array should be at any point in time."

Assuming that ``tasks`` will be an array filled with ``Task`` items, we can work on that ``return`` statement again.
We want to return a list of divs, one for each task.
We can do this within the ``<div className="App"></div>`` container div, using curly braces to run vanilla javascript within the context of a rendering component.

.. code-block:: javascript

    const App: FunctionComponent = () => {
        const [ tasks, setTasks ] = useState<Array<Task>>([]);
        return (
            <div className="App">
                { tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>) }
            </div>
        );
    }

Here we're taking the body of every task and using it as content for the ``<div>`` that contains it.
We're also taking the ``_id`` of each task and using that unique identifier as the "key" of each ``<div>``.
This allows React to keep track of each ``<div>``, which will come in handy when we start mutating the state of this list on the fly.
We're taking all those divs, however many there may be, and making them children of ``<div className="App">``.
Considering the 5 tasks we currently have in our back-end, we would expect the resulting HTML to appear as:

.. code-block:: html

    <div className="App">
        <div>Go grocery shopping</div>
        <div>Go to BodyPump on Friday</div>
        <div>Clean the bathroom</div>
        <div>Change the Brita filter</div>
        <div>Meet Marcus for lunch</div>
    </div>

Commit!

.. code-block:: shell

    [list-tasks] $ git add src/App.tsx
    [list-tasks] $ git commit -m 'Updated the App component to be stateful with respect to the array of Tasks. Also added the list of divs, one for each task, to the returned and rendered DOM'

With that set, we have to actually fetch the data we'll be needing from the back-end.
We could use the built-in ``fetch`` API, but I prefer ``axios``.
In my opinion, it's simpler to retrieve JSON data from an ``axios`` call.
We can install ``axios`` like any other node package

.. code-block:: shell

    $ npm install axios

and include it in our codebase to fetch data like so:

.. code-block:: javascript

    const App: FunctionComponent = () => {
        const [ tasks, setTasks ] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = 'http://localhost:5000/api/v1/tasks';
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        return (
            <div className="App">
                { tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>) }
            </div>
        );
    }

Axios can be used as a Promise, or can be used with ``await`` inside an ``async`` function.
We're using it with ``await``, which means for us that when ``fetchTasks`` is called, this function will wait for a response from the server, then continue to act once the response is retrieved.

Commit!

.. code-block:: shell

    [list-tasks] $ git add src/App.tsx
    [list-tasks] $ git commit -m 'Added the fetchTasks function to the App component. Ready to fetch tasks on one function call.'

Now we have this function added to our component.
If called, it'll send a request to ``'http://localhost:5000/api/v1/tasks'`` and use the data it gets back to update the state of the array of Tasks.
The way it's written, though, it'll do nothing and never get called.

I'm going to add a temporary little button that'll just ensure that this call works the way it's supposed to work and that the DOM is populated accordingly.
In the ``return`` statement I'll put this button within the App div.

.. code-block:: javascript

    <div className="App">
        <button onClick={ () => fetchTasks() }>Click me!</button>
        { tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>) }
    </div>

Let's do the important thing and make sure this all works.
Turn on the Flask server with ``flask run`` in the ``todo-list-server`` directory, and run the front-end with ``npm start`` in the ``todo-list-client`` directory.
Visit the client in your browser and you should see only the button saying "Click me!".

CLICK IT!

If everything is wired together as it should be, clicking the button should fire a ``GET`` request to the server at ``localhost:5000/api/v1/tasks``, retrieve the only 5 tasks we're serving, set them as the array of tasks in our client, and populate the DOM with 5 divs containing the body of each task.

Note: clicking the button again will not add *more* tasks to the task list.
The button's only effect is to retrieve as many task items as the server has to give, and set them to be the array of tasks in the ``App`` component.

Moving forward, we don't want to trigger the loading of task items in this way, so let's remove the button.
We want the tasks to be loaded as soon as the page loads.
For this, we can use React's ``useEffect`` hook.
If you're familiar at all with the ``componentDidMount`` and ``componentDidUpdate`` class methods, this hook replaces them.
Its job is to fire when any part of the component changes state.
It takes two arguments:

- The function that is to run when the component state updates
- An array of stateful objects to be watched, firing the function in the first argument if any of them update

We'll include it in our codebase like so:

.. code-block:: javascript

    useEffect(() => {
        fetchTasks();
    }, []);

What this says is "when the component first loads, call ``fetchTasks``".
After that, never call this function again since nothing is being watched for an updated state.

The whole ``App.tsx`` file should now look like this in its entirety:

.. code-block:: javascript

    import React, { FunctionComponent, useState, useEffect } from 'react';
    import { Task } from './types';
    import axios from 'axios';
    import './App.css';

    const App: FunctionComponent = () => {
        const [tasks, setTasks] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = 'http://localhost:5000/api/v1/tasks';
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        useEffect(() => {
            fetchTasks();
        }, []);

        return (
            <div className="App">
                {tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>)}
            </div>
        );
    }

    export default App;

As you should always do, run the client to make sure that all is working as it should.
Then commit and merge to master.

.. code-block:: shell

    [list-tasks] $ git add src/App.tsx
    [list-tasks] $ git commit -m 'First view done; tasks can be retrieved and listed on the front-end.'
    [list-tasks] $ git checkout master
    [list-tasks] $ git merge list-tasks -m 'Merging update to front end view listing tasks on the page'

Adding the Database
-------------------

We have a working front-end, and that's all well and good.
But right now we're just serving fake data.
Let's allow ourselves to have *real* data.
Let's use our back-end to talk to `MongoDB <https://flask-pymongo.readthedocs.io/en/latest/>`_.

Within ``todo-list-server`` we'll install ``flask-pymongo`` and add it to our ``requirements.txt``.

.. code-block:: shell

    (ENV) [master] $ git checkout -b add-database
    (ENV) [add-database] $ pip install flask-pymongo==2.2.0
    (ENV) [add-database] $ echo 'flask-pymongo==2.2.0' >> requirements.txt

We're also going to want to actually install MongoDB itself.
PyMongo doesn't do that for us, it only installs the client that allows Python to talk to a running instance of MongoDB.

On OSX we can install and run MongoDB with Homebrew

.. code-block:: shell

    (ENV) [add-database] $ brew install mongodb-community@4.0
    (ENV) [add-database] $ brew services start mongodb/brew/mongodb-community

On Ubuntu we can install and run MongoDB with ``apt`` (from the MongoDB docs `here <https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/>`_)

.. code-block:: shell

    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
    $ echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    $ sudo apt-get update
    $ sudo apt-get install -y mongodb-org=4.0.9 mongodb-org-server=4.0.9 mongodb-org-shell=4.0.9 mongodb-org-mongos=4.0.9 mongodb-org-tools=4.0.9
    $ sudo service mongod start

When installed locally, MongoDB will by default run on port ``27017``.

Before we add PyMongo to our server, let's take a second to understand how it can be used.
Open the Python prompt and import the ``MongoClient`` from ``PyMongo``.

.. code-block:: python

    >>> from pymongo import MongoClient

In order to do anything of value, the ``MongoClient`` needs to connect to something.
We tell it what to connect to by providing a mongo URI.

.. code-block:: python

    >>> client = MongoClient("mongodb://localhost:27017/")

With the client now active, we can access any existing database within our running MongoDB instance.
We can also create new ones just by naming them.

.. code-block:: python

    >>> db = client.tasksdb

Every Mongo database is organized into "collections", with every collection containing "documents".
Each "document" is effectively an object that we want to store in our database.
Previously, we created 5 fake tasks which had properties of "_id", "body", "complete", and "creationDate".
Each one of these will be a task "document" and live in the collection of task documents conveniently titled "tasks".

.. code-block:: python

    >>> EXAMPLE_TASKS = [
    ...         {"body": "Go grocery shopping", "complete": False, "creation_date": '03 April 2019 08:33:04 UTC'},
    ...         {"body": "Go to BodyPump on Friday", "complete": False, "creation_date": '08 April 2019 12:41:21 UTC'},
    ...         {"body": "Clean the bathroom", "complete": False, "creation_date": '11 April 2019 16:03:35 UTC'},
    ...         {"body": "Change the Brita filter", "complete": False, "creation_date": '15 April 2019 13:22:11 UTC'},
    ...         {"body": "Meet Marcus for lunch", "complete": False, "creation_date": '27 April 2019 04:27:59 UTC'},
    ...     ]

Let's insert these fine tasks into our database.
Note that I've removed their "_id" fields since that's actually generated by MongoDB on insertion.

.. code-block:: python

    >>> db.tasks.insert_one(EXAMPLE_TASKS[0])
    <pymongo.results.InsertOneResult object at 0x1103bafc8>

Hey, we have our first document inserted into our database!
Let's retrieve it and see how it looks (I assure you there won't be any surprises).

.. code-block:: python

    >>> tasks = [task for task in db.tasks.find()]
    >>> print(tasks)
    [{'_id': ObjectId('5cc7e23bb01fa85d60d6a89b'), 'body': 'Go grocery shopping', 'complete': False, 'creation_date': '03 April 2019 08:33:04 UTC'}]

We'll explore more of the PyMongo functionality soon, but at least now we've seen how to add new items to our Mongo database and access them after they've been inserted.

Let's include PyMongo in ``app.py`` through an import.

.. code-block:: python

    from flask_pymongo import PyMongo

We configure PyMongo by first supplying the Flask app instance with a ``MONGO_URI`` that PyMongo will read.
Then we get access to the Mongo API by creating an instance of the PyMongo client.

.. code-block:: python

    app.config['MONGO_URI'] = "mongodb://localhost:27017/tasksdb"
    mongo = PyMongo(app)

PyMongo will look for that ``MONGO_URI`` and use it to manage the connection to the database whether it's local or remote.

Now, instead of importing our fake data and returning that to our client, we can modify our ``get_tasks`` endpoint function to access the MongoDB database and pull out whatever tasks are stored.
That function can now look like this:

.. code-block:: python

    @app.route("/api/v1/tasks", methods=["GET"])
    def get_tasks() -> list:
        """The home route.

        This view serves data that'll be consumed by the React client.

        Returns
        -------
        list
            A list of incomplete tasks
        """
        tasks = mongo.db.tasks.find({'complete': False})
        return tasks

The call to the database is in ``mongo.db.tasks.find({'complete': False})``, which effectively says:

1. Access the DB that the client is pointing to
2. Access the "tasks" collection within that database
3. Within the "tasks" collection, retrieve all documents that have the property of "complete" set to False

At this point, our ``app.py`` should look like the following:

.. code-block:: python

    from flask_api import FlaskAPI
    from flask_cors import CORS
    from flask_pymongo import PyMongo

    app = FlaskAPI(__name__)
    app.config["MONGO_URI"] = 'mongodb://localhost:27017/tasksdb'
    mongo = PyMongo(app)
    CORS(app)

    @app.route("/api/v1/tasks", methods=["GET"])
    def get_tasks() -> list:
        """The home route.

        This view serves data that'll be consumed by the React client.

        Returns
        -------
        list
            A list of incomplete tasks
        """
        tasks = mongo.db.tasks.find({'complete': False})
        return tasks

Let's commit, run our server, run our client, and witness the magic.

.. code-block:: shell

    (ENV) [add-database] $ git add app.py requirements.txt
    (ENV) [add-database] $ git commit -m 'The get_tasks endpoint now actually returns tasks from the database.'
    (ENV) [add-database] $ flask run

Visiting the client in the browser will trigger a request for the tasks from the ``get_tasks`` route.

That call *will fail*.

The reason why it fails is that the call to ``mongo.db.tasks.find`` doesn't actually return a list of values.
It returns a "Cursor" object, which effectively is a pointer to the set of items you requested willing to show us those items when we ask for them.
We can ask for them by turning them into a list.
Let's amend our function.

.. code-block:: python

    def get_tasks() -> list:
        """The home route.

        This view serves data that'll be consumed by the React client.

        Returns
        -------
        list
            A list of incomplete tasks
        """
        tasks = [task for task in mongo.db.tasks.find({'complete': False})]
        return tasks

Run the Flask server again and access the client home page.
Oh no! Another, different error!
At the end of a sizeable stack trace we'll see:

.. code-block:: shell

    TypeError: Object of type ObjectId is not JSON serializable

Recall that any endpoint function in Flask must return something that's JSON serializable.
That narrows down the possibilities a lot: strings, numbers, booleans, lists, and dictionaries.
If we look at what was actually returned from our database before, we saw this

.. code-block:: shell

    [{'_id': ObjectId('5cc7e23bb01fa85d60d6a89b'), 'body': 'Go grocery shopping', 'complete': False, 'creation_date': '03 April 2019 08:33:04 UTC'}]

Where the ``_id`` of the object might appear at first glance to be a string, but is in fact a BSON (Binary JSON) Object ID.
Mongo uses these BSON object IDs internally to streamline lookup for individual documents.

Fortunately for us, we can convert it into a string fairly simply.
Let's amend the ``get_tasks`` function.

.. code-block:: python

    def get_tasks() -> list:
        """The home route.

        This view serves data that'll be consumed by the React client.

        Returns
        -------
        list
            A list of incomplete tasks.
        """
        results = mongo.db.tasks.find({'complete': False})
        tasks = []
        for task in results:
            task["_id"] = str(task["_id"])
            tasks.append(task)
        return tasks

Now our front-end can consume as many tasks as it would like from our server.
Time to commit and merge.

.. code-block:: shell

    (ENV) [add-database] $ git add app.py 
    (ENV) [add-database] $ git commit -m 'Connected the mongodb client and updated the get_tasks function to serve data from the database instead of directly from file.'
    (ENV) [add-database] $ git checkout master
    (ENV) [master] $ git merge add-database -m 'Merging the addition of the Mongo database'

Store New Data Server-Side
--------------------------

Now that we have somewhere to actually persist data, let's add to our server an endpoint that will receive and store that data.
Checkout a new branch on the Flask server and let's get to work.

.. code-block:: shell

    (ENV) [master] $ git checkout -b new-task

The procedure here will be as follows: 

- receive data from some client that will be the new task
- set the creation date for the new task in the moment of reception
- insert the new task into the ``tasks`` database
- return the newly-inserted task to the client for confirmation

Let's start by creating the endpoint within ``app.py`` that'll be used to collect and store the incoming data.

.. code-block:: python

    # at the top
    from flask_api import FlaskAPI, status

    @app.route("/api/v1/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        return {}, status.HTTP_201_CREATED

Here we're saying that whenever a request is sent to the ``/api/v1/tasks`` route via a ``POST`` request, this function will handle it.
It doesn't matter that the URI for this route is the same as the first route we created; that one handles ``GET`` requests and this handles ``POST`` requests.
Never shall the two be confused by the server.
Also note how we don't have to do any work to manage cross-origin resource sharing.
Flask CORS handles that for us, so we can focus instead on "business logic".
Finally, we leverage FlaskAPI's ``status`` object to dictate the status code of the server's response.
Yes, we could just go with a standard ``200 OK`` status code, but we can be a bit more explicit with what happened if we provide a ``201 Created``.

Now we need to gather our data.
When we're doing a simple thing like serving data we already have on the server to the client, we don't need to access any information about the incoming request.
However, now that we're receiving data from the client we'll need to access the incoming request, as that request holds the data that we need to operate.

Unlike many Python web frameworks, with Flask the incoming request is a global object that must be imported.
It's populated per incoming request, but it exists outside of the functions that access it.
We gain access to it like so.

.. code-block:: python

    from flask import request

Yes, we could've done this before, but we didn't need it yet.

When there's incoming data to be accessed and used within our codebase, it's available through the ``request`` object's ``data`` property.
Within our endpoint function, it'll look like so:

.. code-block:: python

    @app.route("/api/v1/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        new_task = request.data
        return {}, status.HTTP_201_CREATED

When we insert the data into our database, we need it to contain the following fields: ``_id``, ``body``, ``complete``, and ``creationDate``.
The client will only be sending the ``body`` and the ``complete`` status (which will always be False).
MongoDB will populate the ``_id`` field for us.
We have to come up with the ``creationDate`` ourselves.
We need to work with dates and times, just a bit.

Unless you are a unique individual that has *years* of experience working with dates and times in codebases, you should **NEVER** roll your own datetime software.
More likely than not, you will not be able to account for all the various nuances that come with working with dates and times.
Instead, leverage what the language gives you and always set your times as Coordinated Universal Time (UTC) to avoid timezone problems.
If you need timezone support, gather the UTC offset from the client's browser and use that to correct for local time.

Python's ``datetime`` library will allow us to do all the date and time work we'll need to get started.
And, it already comes prepackaged within the standard library, so no need for another ``pip install``!
Let's open the Python prompt and see how it can work for us.

.. code-block:: python

    >>> from datetime import datetime

When a new task is being created, we want to set its creation date and time to that moment immediately before entry into the database.
``datetime.utcnow()`` allows us to get that instant of time, automatically set to UTC.

.. code-block:: python

    >>> datetime.utcnow()
    datetime.datetime(2019, 5, 1, 5, 11, 48, 920374)

Calling ``datetime.utcnow()`` returns a ``datetime`` object, with all the information about the date and time when it was called.
``datetime`` objects are not JSON serializable by default, so we should seek to store that date and time as a string for easy retrieval and easy transmission.
The ``strftime`` method on datetime objects allows us to retrieve the string version of that object, set to whatever format we desire.
I like to think of ``strftime`` as saying "string format time", or "format my time as a string".
Check the Python strftime reference [#f3]_ for details on how you can format that time.

I've found that a datetime format that's parseable with minimal intervention on both the Python and JavaScript sides of applications is ``'%d %B %Y %H:%M:%S UTC'``.
Here's what these arcane symbols translate to:

- ``%d``: two-digit day (e.g. 01)
- ``%B``: full name of the month (e.g. October)
- ``%Y``: four-digit year (e.g. 2019)
- ``%H``: two-digit hour (e.g. 05)
- ``%M``: two-digit minute (e.g. 20)
- ``%S``: two-digit seconds (e.g. 25)

Anything that's not preceded by ``%`` is interpreted literally, including whitespace.
We can see that in our Python prompt.

.. code-block:: python

    >>> now = datetime.utcnow()
    >>> time_fmt = '%d %B %Y %H:%M:%S UTC'
    >>> now.strftime(time_fmt)
    '01 May 2019 05:23:59 UTC'

Let's incorporate this into ``app.py``.
We want to set the creation date as soon as we receive the data, and add that creation date to the data.

.. code-block:: python

    # underneath all the package imports
    TIME_FMT = '%d %B %Y %H:%M:%S UTC'

    # back to the endpoint
    @app.route("/api/v1/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        new_task = request.data
        now = datetime.utcnow()
        new_task["creationDate"] = now.strftime(TIME_FMT)
        return {}, status.HTTP_201_CREATED

At this point, our data has the task body, completion status, and creation date.
What we need next is to insert it into MongoDB.
This is pretty straightforward, as there's an ``insert_one`` method on the MongoDB collection!

.. code-block:: python

    @app.route("/api/v1/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        new_task = request.data
        now = datetime.utcnow()
        new_task["creationDate"] = now.strftime(TIME_FMT)

        mongo.db.tasks.insert_one(new_task)
        return {}, status.HTTP_201_CREATED

An interesting thing happens when ``insert_one`` is called.
The dict that it was called on, if it has been assigned to a variable, picks up the ID granted by the database as a key-value pair.
The key, as we've seen previously, will be ``"_id"``, and the value will be a BSON ObjectId type object.
We want to return our newly-inserted object to the client, but we can't do so as long as that field's value is a BSON ObjectId instead of a string.

.. code-block:: python

    @app.route("/api/v1/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        new_task = request.data
        now = datetime.utcnow()
        new_task["creationDate"] = now.strftime(TIME_FMT)

        mongo.db.tasks.insert_one(new_task)
        new_task["_id"] = str(new_task["_id"])
        return new_task, status.HTTP_201_CREATED

Now our task creation endpoint is ready to be accessed by our client!
Let's commit and merge our code, then flip back to the client to update our view and submit data.

.. code-block:: shell

    (ENV) [new-task] $ git add app.py
    (ENV) [new-task] $ git commit -m 'Added the new_task route. Now the server can insert into the database'
    (ENV) [new-task] $ git checkout master
    (ENV) [master] $ git merge new-task -m 'Merging the addition of the new task route'

Data Submission, Controlling Components, and Rethinking Structure
-----------------------------------------------------------------

Now that we have our ``POST`` endpoint, we need to enable the client to actually collect and send user input to our server.
We're going to need a form.
Let's start a new branch in the client application for adding a submission form.

.. code-block:: shell

    [master] $ git checkout -b add-tasks

We can start by creating a classic ``<form>`` element containing a ``<textarea>`` and a submission button within the ``return`` statement of our existing ``src/App.tsx``.
We're going to use placeholder text within the ``<textarea>`` instead of a label for the form, but we could have a label if we so chose.
.. code-block:: javascript

    import React, { FunctionComponent, useState, useEffect } from 'react';
    import { Task } from './types';
    import axios from 'axios';
    import './App.css';

    const App: FunctionComponent = () => {
        const [tasks, setTasks] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = 'http://localhost:5000/api/v1/tasks';
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        useEffect(() => {
            fetchTasks();
        }, []);

        return (
            <div className="App">
                <form>
                    <textarea placeholder="What do you want to do?" />
                    <button type="submit">Add Task</button>
                </form>
                {tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>)}
            </div>
        );
    }

    export default App;

The way that this is written, the form will do nothing.
Nothing at all--we won't even be able to type in the ``<textarea>``.
Understanding why helps this make sense.

When React renders your components on the page, it renders them in whatever way was dictated by the props and states of those components.
When we created the form, we effectively said that the text field will contain no text, and the submit button will do nothing of real value.

Inserting text into a ``<textarea>`` or ``<input>`` field involves an event.
Specifically an "onChange" event, where we could make something happen when the content of the element is being triggered to change.
What we're used to--what we've always taken for granted--is that when we press a key, the text field should update with the value of that key.
With React, however, you have to explicitly make that connection.

We're going to want to do something like...

.. code-block:: javascript

    var taskBody = '';

    <textarea
        placeholder="What do you want to do?"
        value={ taskBody }
        onChange={ (event) => { taskBody = event.target.value; }}
    />

Here we say that the ``<textarea>`` element gets what it's about to display from the ``textContent`` variable.
If we change ``textContent`` in a way that respects the current and future state of the component, then the value of that element should populate with the change.

There's a problem here though.
When ``textContent`` is created in the above example, it isn't stateful.
That means that even though the value of ``textContent`` might be updated, the user doesn't see it because the component doesn't rerender.
We need the form to maintain its own state so that we can update this value as we change it.

It's time to break off into another component.
More than that though, it's time for a bit of a reorganization of the codebase.

In the ``src`` directory, currently we have just our ``App.tsx`` which contains the whole app, our ``index.tsx`` which imports the app, and the corresponding ``.css` files.
We want to start to rethink ``App.tsx``, re-envisioning it as being the collection point for all the components of our application instead of housing all the raw code that comprises the application.
To move ``App.tsx`` toward this new purpose, we're going to give ourselves a rule: **any HTML element that isn't a containing element gets imported as a whole component instead of being written out**.
``App.tsx`` will then just hold these components and be focused on managing any of the global state.

Make a new directory within ``src`` called ``components``. 
This new directory will house all the components we build.
Within that directory create two ``.tsx`` files:

- ``TaskList.tsx``: will house the list of tasks
- ``CreateTask.tsx``: will house the form for creating new tasks

.. code-block:: shell

    $ mkdir src/components
    $ touch src/components/{TaskList,CreateTask}.tsx

We're going to first move the list of tasks from ``App.tsx`` into ``components/TaskList.tsx``, then import the ``TaskList`` component.
The ``TaskList`` component that gets exported will take the array of ``Task`` items as its props.

.. code-block:: javascript

    // in components/TaskList.tsx
    import React from 'react';
    import { Task } from '../types';

    interface Props {
        tasks: Task[];
    }

    export const TaskList = ({ tasks }: Props) => {
        return <div>
            {tasks.map((task: Task) => <div key={ task._id }>{ task.body }</div>)}
        </div>;
    }

Here we defined the ``interface`` for the ``TaskList`` component, listing each prop that it'll take in along with the expected object type.
We're specifying just an array of Tasks for now.

We then create the ``TaskList`` component, seeking to export it and make it available by name on import.
We set the parameter list to be of the type we declared above, and actually include the parameters we expect by name.
Then, we just return the same list of tasks we had previously written out in ``App.tsx``.

Having built this component, let's commit.

.. code-block:: shell

    [add-tasks] $ git add src/components/TaskList.tsx
    [add-tasks] $ git commit -m 'Moved the list of tasks into its own component.'

With this change made, we can go back to ``App.tsx``, remove the list of tasks, and replace it with the ``TaskList`` component.

.. code-block:: javascript

    // in App.tsx
    import React, { FunctionComponent, useState, useEffect } from 'react';
    import axios from 'axios';
    
    import { TaskList } from './components/TaskList';
    
    import { Task } from './types';

    import './App.css';

    const App: FunctionComponent = () => {
        const [tasks, setTasks] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = 'http://localhost:5000/api/v1/tasks';
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        useEffect(() => {
            fetchTasks();
        }, []);

        return (
            <div className="App">
                <form>
                    <textarea placeholder="What do you want to do?" />
                    <button type="submit">Add Task</button>
                </form>
                <TaskList tasks={ tasks } />
            </div>
        );
    }

We replaced the list of tasks with the ``TaskList`` component, passing as props the ``tasks`` array.
When it renders, it'll render exactly as it had before.

Some other smaller changes were made in this file.
Since we'll be importing a couple more things into this file, it helps to have a bit of structure in the import list.
The way I like to section my imports is the following order, with each section separated by a single blank line for readability:

1. External libraries
2. Utilities
3. Components
4. Types
5. Stylesheets

This isn't law, it's just how I do it.
There is no one "right" way.

Let's do the same thing we did with the task list to the task creation form.
It'll be even simpler to start: the form takes no props!

.. code-block:: javascript

    // in components/CreateTask.tsx
    import React from 'react';

    export const CreateTask = () => {
        return <form>
            <textarea placeholder="What do you want to do?" />
            <button type="submit">Add Task</button>
        </form>;
    }

Let's import it into ``App.tsx`` then get to work on adding its own internal component state.

.. code-block:: javascript

    // in App.tsx
    import React, { FunctionComponent, useState, useEffect } from 'react';
    import axios from 'axios';
    
    import { TaskList } from './components/TaskList';
    import { CreateTask } from './components/CreateTask';
    
    import { Task } from './types';

    import './App.css';

    const App: FunctionComponent = () => {
        const [tasks, setTasks] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = 'http://localhost:5000/api/v1/tasks';
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        useEffect(() => {
            fetchTasks();
        }, []);

        return (
            <div className="App">
                <CreateTask />
                <TaskList tasks={ tasks } />
            </div>
        );
    }

Alright let's focus in on this ``CreateTask`` component.

.. code-block:: javascript

    export const CreateTask = () => {
        return <form>
            <textarea placeholder="What do you want to do?" />
            <button type="submit">Add Task</button>
        </form>;
    }

The first thing we want to be able to do is have our ``<textarea>`` actually be able to take and display user input as that input arrives.
The way we do this is with a stateful variable representing that input.
For that we bring out ``useState`` again.

.. code-block:: javascript

    import React, { useState } from 'react';

    export const CreateTask = () => {
        const [ taskBody, setBody ] = useState('');

        return <form>
            <textarea placeholder="What do you want to do?" />
            <button type="submit">Add Task</button>
        </form>;
    }

We expect the ``taskBody`` to be a string and for the text area to have nothing in it besides the placeholder text to start, so we initialize it as an empty string.
Whenever a user types within the ``textarea``, we want to update the state of this string, so we'll have the "change" event trigger a call to ``setBody``.

.. code-block:: javascript

    import React, { useState } from 'react';

    export const CreateTask = () => {
        const [ taskBody, setBody ] = useState('');

        return <form>
            <textarea
                placeholder="What do you want to do?"
                value={ taskBody }
                onChange={ event => setBody(event.target.value) }
            />
            <button type="submit">Add Task</button>
        </form>;
    }

Fire up the client app to make sure this works.
Woohoo! Now our textarea actually updates.
Let's actually enable ourselves to submit something.

Conceptually, when the user has written out everything they want in the ``<textarea>``, we want to be able to gather that content and submit a POST request to the server.
We then want to be able to update the overall task list.

That last bit is key.
Because we want this component to effectively update the state of the application outside of itself, it'll need some access to a function that manages that state.
Because we've broken this form out into its own component, we'll need to pass that function into the component as props from its parent, ``<App />``

Within the ``App`` component, we're going to create a function that'll handle the submission of that data to the server.
We won't fill it out quite yet, but we will decide on what data it'll be taking in.

.. code-block:: javascript

    // in App.tsx
    // within the App component
    async function submitTask(body: string) {
        console.log(body);
    }

    // within the return statement
    <CreateTask submitTask={ submitTask } />

Now that we know the ``submitTask`` function is coming in as props to ``CreateTask``, let's create an interface that'll expect this function, and update the component to use both the interface and the function.

.. code-block:: javascript

    // in components/CreateTask.tsx
    import React, { useState } from 'react';

    interface Props {
        submitTask: (body: string) => void;
    }

    export const CreateTask = ({ submitTask }: Props) => {
        const [taskBody, setBody] = useState('');

        return <form onSubmit={ () => submitTask(taskBody) }>
            <textarea
                placeholder="What do you want to do?"
                value={taskBody}
                onChange={event => setBody(event.target.value)}
            />
            <button type="submit">Add Task</button>
        </form>;
    }

Here, when the form is submitted, whatever the current value of ``taskBody`` happens to be will get passed to the ``submitTask`` function.
However, there are two issues that we've now introduced.

1. A form submission event triggers a page load every time. We don't want that to happen, we just want to submit the data without interrupting the user experience
2. When the form is submitted (and the page isn't reloaded) the value of the ``textarea`` still contains the old content. Ideally, when the data has been sent for submission, the field would clear.

If we want to address both of those issues, we're going to have to make the function called in the event of a submission a little more complex.
In the interest of not cluttering the HTML elements, let's create a function within ``CreateTask`` to handle what we need.
Then, we'll call that function when the form is submitted.

.. code-block:: javascript

    // in components/CreateTask.tsx
    import React, { FormEvent, useState } from 'react';

    interface Props {
        submitTask: (body: string) => void;
    }

    export const CreateTask = ({ submitTask }: Props) => {
        const [taskBody, setBody] = useState('');

        const handleSubmission = (event: FormEvent<HTMLFormElement>) => {
            event.preventDefault();
            submitTask(taskBody);
            setBody('');
        }

        return <form onSubmit={ handleSubmission }>
            <textarea
                placeholder="What do you want to do?"
                value={taskBody}
                onChange={event => setBody(event.target.value)}
            />
            <button type="submit">Add Task</button>
        </form>;
    }

The ``handleSubmission`` function encapsulates what we need quite nicely.
First, note the type of object coming in on the parameters.
We're expecting a form submission event, so it's type ``FormEvent<HTMLFormEvent>``, where ``FormEvent`` is imported from React and ``HTMLFormEvent`` is a built-in.
Then we have ``event.preventDefault()`` to cut off the page load.
We keep our same call to ``submitTask``, passing it the current state of ``taskBody``.
Immediately afterward, we set the ``taskBody`` value to an empty string, preparing the ``textarea`` for whatever the next input might be.

We can leave this component alone for now.
Let's commit then go back to ``App.tsx`` and actually handle the submission of data.

.. code-block:: shell

    [add-tasks] $ git add src/components/CreateTask.tsx
    [add-tasks] $ git commit -m 'Built out the form that will create the body of the new task.'

Currently, ``submitTask`` is effectively empty, only serving to log the data to the browser's console.
Let's change that.
We want this function to submit a POST request to our server's endpoint ``/api/v1/tasks``.
We want this request to contain the body of our new task item, as well as set its completion status to ``false``.
Finally, once the request has been submitted and received, we want to retrieve the full task item from the server, and add it to the list of tasks.

One more detail that'll influence how we write the function.
The hostname for the server being accessed by the ``GET`` request is the same as the one being accessed by the ``POST`` request, and will be the same for the upcoming ``PUT`` and ``DELETE`` requests.
Instead of writing it out every time, why don't we write it once and use template literals?

.. code-block:: javascript

    import React, { FunctionComponent, useState, useEffect } from 'react';
    import axios from 'axios';

    import { TaskList } from './components/TaskList';
    import { CreateTask } from './components/CreateTask';

    import { Task } from './types';

    import './App.css';

    const API_HOST = 'http://localhost:5000/api/v1'

    const App: FunctionComponent = () => {
        const [tasks, setTasks] = useState<Array<Task>>([]);

        async function fetchTasks() {
            const apiUrl: string = `${API_HOST}/tasks`;
            const result = await axios.get(apiUrl);

            setTasks(result.data);
        };

        async function submitTask (body: string) {
            const apiUrl: string = `${API_HOST}/tasks`;
            const newTask = {
                body,
                complete: false
            }
            const result = await axios.post(apiUrl, newTask);

            setTasks(tasks.concat(result.data));
        }

        useEffect(() => {
            fetchTasks();
        }, []);

        return (
            <div className="App">
                <CreateTask submitTask={ submitTask } />
                <TaskList tasks={tasks} />
            </div>
        );
    }

    export default App;

The constant ``API_HOST`` declared outside ``App`` gets incorporated into ``fetchTasks`` and ``submitTask`` as the base of the each function's ``ApiUrl``.

``submitTask`` gets converted to an ``async`` function and filled out as

.. code-block:: javascript

    async function submitTask (body: string) {
        const apiUrl: string = `${API_HOST}/tasks`;
        const newTask = {
            body,
            complete: false
        };
        const result = await axios.post(apiUrl, newTask);

        setTasks(tasks.concat(result.data));
    }

The data to be transmitted becomes the second argument of ``axios.post``.
The return value is gathered, and the data that was sent back within the server's response gets added to the task list and used to update the state of the task list.
Note that ``tasks.concat`` will create a copy of the existing task list and add the new data to that copy, using that copy as the new state of the task list.

And now we have our post request!
We can add as many tasks as we like!
Let's commit, merge to master, then work out how to remove tasks.

.. code-block:: shell

    [add-tasks] $ git add src/components/App.tsx
    [add-tasks] $ git commit -m 'Filled out the submitTask function within App.tsx, and imported all the necessary components'
    [add-tasks] $ git checkout master
    [master] $ git merge add-tasks -m 'Merging the creation of the components subdirectory and the CreateTask form.'

Deleting Items, Back to Front
-----------------------------

Let's create a new branch and start at the server.

.. code-block:: shell

    (ENV) [master] $ git checkout -b remove-task

The endpoint that'll remove task items from the database really just needs to serve as a wrapper around the PyMongo method to delete items.
Our Mongo client will do the trick with the ``mongo.db.<collection name>.delete_one()`` method.
This method will take some criteria to match against all the documents within that collection, deleting one that matches.

In our situation, we can guarantee the uniqueness of a task's ID, so we can provide that to ``delete_one()`` and it'll handle the rest.
When we provide the ID, we need to make sure that what we're providing is a BSON  ``ObjectId``, not simply a string.
The database stores the ID as BSON.

Because we're trying to adhere to building a REST-like interface, the route that directs traffic to our endpoint will take only ``"DELETE"`` requests, with the task's id in the URL.

.. code-block:: python

    # in app.py
    @app.route('/api/v1/tasks/<task_id>', methods=["DELETE"])

Let's refactor this route, and actually all the routes, a little bit.
Note that they all share the same base: ``"/api/v1"``.
Just like we did with the front end, let's move that common base to one variable and use that variable in a variety of places with string formatting.

.. code-block:: python

    ... import stuff ...

    ... app setup stuff ...

    TIME_FMT = '%d %B %Y %H:%M:%S UTC'
    API_PREFIX = "/api/v1"

    @app.route(f"{API_PREFIX}/tasks", methods=["GET"])
    def get_tasks() -> list:
        ... the first endpoint ...

    @app.route(f"{API_PREFIX}/tasks", methods=["POST"])
    def new_task() -> dict:
        ... the second endpoint ...

    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["DELETE"])

Now let's continue with our endpoint function.
Notice this route looks a little different from the other routes.
With Flask, when we want a route to capture variables, we indicate that with the angle braces around the name of the variable.
Then, when we declare the function that will respond to the route, we add that same exact variable name into the function's parameter list.
By default, any variable in a route is interpreted as a string.
If we had an integer it'd be ``"<int:task_id>"``, and a float would be ``"<float:task_id>"``.

Since the purpose of this function is to delete one task by ID, let's call it ``delete_task``.
When that task is deleted, the function will return the string representation of the object's ID to the client.

.. code-block:: python

    # at the top
    from bson.objectid import ObjectId

    # at the bottom
    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["DELETE"])
    def delete_task(task_id: str) -> str:
        """The Task Deletion route.

        This endpoint takes in the id of the task to be deleted, finds it in the
        database, and removes it. On a successful deletion, it returns the
        id of the deleted task to the client.

        Parameters
        ----------
        task_id : str
            The ID of the task to be deleted, as a string.

        Returns
        -------
        str
            The ID of the deleted task, as a string.
        """
        mongo.db.tasks.delete_one({"_id": ObjectId(task_id)})
        return task_id, status.HTTP_202_ACCEPTED

Here we're telling the Mongo client to search for an item with an ID of whatever we provide, and delete it.
Then we send the ``task_id`` back to the client as proof of deletion, as well as a status code of ``202 Accepted``.
Often you'll find the status code for deletion is ``204 No Content``, however because we're sending content back to the client in the form of the task ID, it'd be disingenuous to use a 204 status code.
We all have our hills to die on.

That's it for deletion.
Let's commit, merge, and make it work in the client.

.. code-block:: shell

    (ENV) [remove-task] $ git add app.py
    (ENV) [remove-task] $ git commit -m 'Created the task deletion route'
    (ENV) [remove-task] $ git checkout master
    (ENV) [master] $ git merge remove-task -m 'Merging in the task deletion endpoint.'

Adding a Delete Button
----------------------

Now that we have our delete route on the server, we need to enable the client to use it.
Let's put a button on each Task item in the task list that'll trigger a deletion on the server and remove that item from the list.

.. code-block:: shell

    [master] $ git checkout -b delete-button

We're going to be adding some complexity to the individual task items.
As such, we should do ourselves a favor and break the task items out into their own component.
Then we can import that component and keep the return value of the ``TaskList`` simple.

.. code-block:: shell

    [delete-button] $ touch src/components/TaskItem.tsx

Let's start the ``TaskItem`` component with the structure we already know.

.. code-block:: javascript

    // components/TaskItem.tsx
    import React from 'react';

    import { Task } from '../types';

    interface Props {
        task: Task;
    }

    export const TaskItem = ({ task }: Props) => {
        return <div>
            { task.body }
        </div>;
    }

Now that we've encapsulated the Task div, we can work independently on the ``TaskItem`` component.
The delete button will be the first of a few buttons that our ``TaskItem`` will need.
As such, we can create a section that'll house all the buttons, and put the delete button there.

.. code-block:: javascript

    export const TaskItem = ({ task }: Props) => {
        return <div>
            <div className="task-buttons">
                <button onClick={ () => {} }>Delete</button>
            </div>
            <div className="task-body">
                { task.body }
            </div>
        </div>;
    }

So what should happen when we click the ``Delete`` button?
As we discussed before, a request should be sent to the server that deletes the whole Task Item from the database.
The user should also no longer see the task item in the browser, and the item should be removed from the array of Tasks.

Since these changes will affect the global application state (i.e. the individual task items that populate the other components of the app), we'll need a function within ``App`` that'll handle the deletion and update the array of Tasks.

Here's the function that'll go into the ``App`` component.

.. code-block:: javascript

    // in App.tsx
    async function deleteTask(taskId: string) {
        const url: string = `${API_HOST}/tasks/${taskId}`;
        await axios.delete(url);
        
        setTasks(tasks.filter((task: Task) => task._id !== taskId));
    }

Here we do the same type of ``axios`` fetch we've done twice already.
Then, we remove the task from the tasks array by returning an array that has filtered out any task with the same ``taskId``.
This is one of the reasons why it's crucial that we use unique values for our task IDs.
If more than one task item shared the same value, that'd also be scrubbed from our tasks array.

Now we have to get this ``deleteTask`` function all the way down into our ``TaskItem`` component to be used by the delete button.
``TaskItem`` lives within the ``TaskList``, so we'll have to pass this function as a prop through ``TaskList`` to get there.

.. code-block:: javascript

    // in the return statement of the App component
    return (
        <div className="App">
            <CreateTask submitTask={ submitTask } />
            <TaskList
                tasks={tasks}
                deleteTask={deleteTask}
            />
        </div>
    );

Let's commit these changes to ``App.tsx``, then keep following ``deleteTask`` as it trickles on down.

.. code-block:: shell

    [delete-button] $ git add src/App.tsx
    [delete-button] $ git commit -m 'Added the deleteTask function that takes a taskId and removes it from the database as well as the front-end store of tasks'

Now that ``TaskList`` is receiving ``deleteTask``, we have to update the props.
We also need to continue passing ``deleteTask`` downward in the component hierarchy to reach the ``TaskItem`` component.

.. code-block:: javascript

    // in components/TaskList.tsx
    interface Props {
        tasks: Task[];
        deleteTask: (taskId: string) => void;
    }

    export const TaskList = ({ tasks, deleteTask }: Props) => {
        return <div>
            {tasks.map((task: Task) => {
                const itemProps = { key: task._id, task, deleteTask };
                return <TaskItem { ...itemProps } />;
            })}
        </div>;
    }

Because the number of props we're passing through to the ``TaskItem`` is starting to get a bit long, I'm choosing to use a little shorthand here so that the return line of the component itself doesn't get too cluttered.
I'm gathering all the props in a separate variable, then using the spread operator ``...`` to provide them separately as arguments to ``TaskItem``.

One more level to go!
Time to commit!

.. code-block:: shell

    [delete-item] $ git add src/components/TaskList.tsx
    [delete-item] $ git commit -m 'Updated the props of TaskList and passed the deleteTask function through to TaskItem. Also collected the props of TaskItem into an object.'

Onward, to the ``TaskItem``!

.. code-block:: javascript

    // in components/TaskItem.tsx
    interface Props {
        task: Task;
        deleteTask: (taskId: string) => void;
    }

    export const TaskItem = ({ task, deleteTask }: Props) => {
        return <div>
            <div className="task-buttons">
                <button onClick={ () => deleteTask(task._id) }>Delete</button>
            </div>
            <div className="task-body">
                {task.body}
            </div>
        </div>;
    }

``deleteTask`` finds its way into the function through its parameters, and gets used right in the ``onClick`` event handler.
Now our ``deleteTask`` function has reached its destination, receiving the ID of the task item as an argument.
Let's fire up the client app and try it out.
Then let's commit, merge, and move on to the server-side "update" functionality.

.. code-block:: shell

    [delete-item] $ git add src/components/TaskItem.tsx
    [delete-item] $ git commit -m 'Added the deleteTask() functionality to the TaskItem.'
    [delete-item] $ git checkout master
    [master] $ git merge -m 'Task items can now be deleted from the client.'

Updating Items in the Database
------------------------------

There's two circumstances within this app where we'd want to update the data held by an individual Task:

- Completion of the task
- Updating the contents of a task

We could make two separate endpoints for each operation, but really we'd be duplicating work.
Instead, we're going to build one endpoint to handle both operations.
The key here?
Have the client side send the whole updated Task Item instead of just a notice to update one field.
Let's make it work.

.. code-block:: shell

    (ENV) [master] $ git checkout -b update-item

Within ``app.py`` we'll declare a new route to be served.

.. code-block:: python

    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["PUT"])

Because we're keeping our app REST-like, the HTTP method we'll use to update an item is "PUT".
The URL will point to a specific task by ID, since we need to target one thing to be changed.

The function that'll attach to this route will use the ``task_id`` from the URL to find and update an item from the database.
The data it'll use to update will come from the incoming request's data.
Once the item is updated, we'll send the updated item back to the client.

.. code-block:: python

    # at the top
    from pymongo import ReturnDocument

    # at the bottom
    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["PUT"])
    def update_task(task_id: str) -> dict:
        """The Task Update route.
        This endpoint takes in the id of the task to be updated in the URL, as
        well as the new state of the task item in the request body. On a
        successful update, it returns the new state of the updated task to the
        client.

        Parameters
        ----------
        task_id : str
            The ID of the task to be updated, as a string.

        Returns
        -------
        dict
            The updated data of the task.
        """
        update_data = request.data
        update_data.pop('_id')
        updated_task = mongo.db.tasks.find_one_and_update(
            {"_id": ObjectId(task_id)},
            {"$set": update_data},
            return_document=ReturnDocument.AFTER
        )
        updated_task["_id"] = str(updated_task["_id"])
        return updated_task

As we saw with the ``"POST"`` request, incoming data can be found within the ``request.data`` object.
We pop the ``"_id"`` field out of the incoming data because we want to be able to just provide the whole object as data to be updated and MongoDB will give us an error if we try to update the ``"_id"`` field.
We could just as easily not pass the ``"_id"`` field from the client, however my own personal feeling is that the client shouldn't be responsible for the shape of the data, only the content.

Updating a MongoDB document is fairly straightforward with use of the ``find_one_and_update`` method.
As the name implies, it finds one item matching your query, then updates it with the data that you pass it.
The first argument passed to ``find_one_and_update`` is the query we use to select the one item to be updated.
The second argument is the data to be set on that item.

Note that you don't have to pass ``find_one_and_update`` every field the object possesses, whether it's being updated or not.
I've chosen to do that here due to simplicity, but we could've just as simply picked out only the data to be changed and used that as the value for ``"$set"``.

The third argument ``return_document`` is optional.
By default, it's set to ``ReturnDocument.BEFORE``, where ``ReturnDocument`` is imported directly from ``PyMongo``.
What this means is that when this function is called, by default it returns to you the original document before modification.
That's not useful for us here, so we use ``ReturnDocument.AFTER`` to retrieve the document after it has been updated.

Why do we do this?
Why not just return the data that came in from the ``request`` instead?
My feeling is that this endpoint should reflect the database's actual state and not its aspirational state.
The incoming data on the ``request`` object is an aspiration.
``ReturnDocument.AFTER`` provides us with the reality, so the client is never confused about what is actually going on server-side.

Finally, we convert the ``"_id"`` of the task item to a string and return it just as we have before with the ``"POST"`` request.

This is the last endpoint we'll need to add to our server!
Here's what it should look like altogether:

.. code-block:: python

    from datetime import datetime
    from bson.objectid import ObjectId

    from flask import request
    from flask_api import FlaskAPI, status
    from flask_cors import CORS
    from flask_pymongo import PyMongo
    from pymongo import ReturnDocument

    app = FlaskAPI(__name__)
    app.config["MONGO_URI"] = 'mongodb://localhost:27017/tasksdb'
    mongo = PyMongo(app)
    CORS(app)

    TIME_FMT = '%d %B %Y %H:%M:%S UTC'
    API_PREFIX = "/api/v1"

    @app.route(f"{API_PREFIX}/tasks", methods=["GET"])
    def get_tasks() -> list:
        """The home route.
        This view serves data that'll be consumed by the React client.

        Returns
        -------
        list
            A list of incomplete tasks, ordered by creation date.
        """
        results = mongo.db.tasks.find({'complete': False})
        tasks = []
        for task in results:
            task["_id"] = str(task["_id"])
            tasks.append(task)
        return tasks


    @app.route(f"{API_PREFIX}/tasks", methods=["POST"])
    def new_task() -> dict:
        """The Task Creation route.

        This endpoint takes in new data that will be constructed into a new
        item for the To Do list.

        Returns
        -------
        dict
            The data of the task, as it has been inserted into the database
        """
        new_task = request.data
        now = datetime.utcnow()
        new_task["creationDate"] = now.strftime(TIME_FMT)

        mongo.db.tasks.insert_one(new_task)
        new_task["_id"] = str(new_task["_id"])
        return new_task

    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["DELETE"])
    def delete_task(task_id: str) -> str:
        """The Task Deletion route.

        This endpoint takes in the id of the task to be deleted, finds it in the
        database, and removes it. On a successful deletion, it returns the
        id of the deleted task to the client.

        Parameters
        ----------
        task_id : str
            The ID of the task to be deleted, as a string.

        Returns
        -------
        str
            The ID of the deleted task, as a string.
        """
        mongo.db.tasks.delete_one({"_id": ObjectId(task_id)})
        return task_id

    @app.route(f"{API_PREFIX}/tasks/<task_id>", methods=["PUT"])
    def update_task(task_id: str) -> dict:
        """The Task Update route.
        This endpoint takes in the id of the task to be updated in the URL, as
        well as the new state of the task item in the request body. On a
        successful update, it returns the new state of the updated task to the
        client.

        Parameters
        ----------
        task_id : str
            The ID of the task to be updated, as a string.

        Returns
        -------
        dict
            The updated data of the task.
        """
        update_data = request.data
        update_data.pop('_id')
        updated_task = mongo.db.tasks.find_one_and_update(
            {"_id": ObjectId(task_id)},
            {"$set": update_data},
            return_document=ReturnDocument.AFTER
        )
        updated_task["_id"] = str(updated_task["_id"])
        return updated_task

The next time we touch this codebase, it'll be to set it up for deployment.
For now though, let's just commit, merge, and concern ourselves with the state of the front-end.

.. code-block:: shell

    (ENV) [update-item] $ git add app.py
    (ENV) [update-item] $ git commit -m 'We can now accept updates to Task Items.'
    (ENV) [update-item] $ git checkout master
    (ENV) [master] $ git merge update-item -m 'The PUT route was added for updating Task Items'

Completing Tasks on the Client Side
-----------------------------------

Completing a task in our ToDo list involves a change in state for that task; an update to the properties of that task.
This will take advantage of our new ``"PUT"`` route.

.. code-block:: shell

    [master] $ git checkout -b complete-task

Let's start at the top of our app, in ``App.tsx``.
The function we'll create will take a whole task item as a parameter and change its completion status to "true", then send an ``axios`` request to our server.
When it receives the updated state of the task, it should remove the task from the array of tasks.

.. code-block:: javascript

    // within the App component
    async function completeTask(task: Task) {
        const url: string = `${API_HOST}/tasks/${task._id}`;
        const updated = Object.assign({}, task, {complete: true});
        await axios.put(url, updated);

        setTasks(tasks.filter((task: Task) => task._id !== updated._id));
    }

We use ``Object.assign`` here because we want to avoid mutating the state of an existing object outside of using some sort of React-generated ``setState`` function.
Instead, we create a new object that copies all the properties of the old object, as well as the new value we want the new object to have.
Like ``axios.post``, ``axios.put`` takes the destination URL as the first argument and the data to be submitted as the second argument.
Finally to remove the completed task from the array of tasks, we just filter out any tasks with a matching ``"_id"``, of which there should be only one.
Let's commit this and move on.

.. code-block:: shell

    [complete-task] $ git add src/App.tsx
    [complete-task] $ git commit -m 'Added functionality to submit a completed task to the server.'

Now that the operation for completing a task has been created, let's roll this into the UI of the task list.
Each individual task item should have the opportunity to be completed.
It'd probably work best if there was some sort of button that could do this.
Easy recognition, quick action.
What this means is we'll need to pass the ``completeTask`` function down to the ``TaskItem`` level.
As such, the ``return`` statement of ``App`` will look like this:

.. code-block:: javascript

    // the return statement of the App component

    const taskListProps = {tasks, deleteTask, completeTask};
    return (
        <div className="App">
            <CreateTask submitTask={submitTask} />
            <TaskList {...taskListProps} />
        </div>
    );

Using that shorthand for passing props again since the number of props going to ``TaskList`` is getting long.
Let's commit and work on passing this function through the ``TaskList``.

.. code-block:: shell

    [complete-task] $ git add src/App.tsx
    [complete-task] $ git commit -m 'Passing the completeTask function through to TaskList'

The ``TaskList`` won't change much.
It'll take our function as one of its props and just pass that through to each ``TaskItem``.

.. code-block:: javascript

    // components/TaskList.tsx
    import React from 'react';
    import { TaskItem } from './TaskItem';
    import { Task } from '../types';

    interface Props {
        tasks: Task[];
        deleteTask: (taskId: string) => void;
        completeTask: (task: Task) => void;
    }

    export const TaskList = ({ tasks, deleteTask, completeTask }: Props) => {
        return <div>
            {tasks.map((task: Task) => {
                const itemProps = { key: task._id, task, deleteTask, completeTask };
                return <TaskItem { ...itemProps } />;
            })}
        </div>;
    }

Update the props interface, update the actual incoming props, and update the props being passed down to ``TaskItem``.
That's the job here, so let's commit it.

.. code-block:: shell

    [complete-task] $ git add src/components/TaskList.tsx
    [complete-task] $ git commit -m 'Passed the completeTask function through to TaskItem'

On the ``TaskItem`` side, we'll need a new button that enacts the ``completeTask`` function.
The function, having finally reached its destination, will take as an argument the ``Task`` that it's modifying.
When the button is clicked, that function gets run.

.. code-block:: javascript

    // in components/TaskItem.tsx
    import React from 'react';

    import { Task } from '../types';

    interface Props {
        task: Task;
        deleteTask: (taskId: string) => void;
        completeTask: (task: Task) => void;
    }

    export const TaskItem = ({ task, deleteTask, completeTask }: Props) => {
        return <div>
            <div className="task-buttons">
                <button onClick={ () => completeTask(task) }>Complete</button>
                <button onClick={ () => deleteTask(task._id) }>Delete</button>
            </div>
            <div className="task-body">
                {task.body}
            </div>
        </div>;
    }

Once again, we update the props interface and update the parameter list of the function.
Finally we add the aforementioned button, give it some descriptive text, and have it call ``completeTask`` when it's clicked.
Let's commit, merge, and move onto the next piece of UI functionality: updating a task's body.

.. code-block:: shell

    [complete-task] $ git add src/components/TaskItem.tsx
    [complete-task] $ git commit -m 'Updated the TaskItem component to enact the completeTask function.'
    [complete-task] $ git checkout master
    [master] $ git merge complete-task -m 'Added the UI functionality to complete a task.'

Updating Tasks on the Client Side
---------------------------------

When we update a task, all that we're really doing in this moment is updating the body of that task.
So any function that sends a request to the server to accomplish this should take in the expected new body, as well as the task itself.
We'll then remove the old version of the task from the tasks array and add the new version, provided by the server.
Let's get this done first, then we'll get to the UI.

.. code-block:: shell

    [master] $ git checkout -b update-task

This function will live, in the same place as the ``completeTask`` function did.

.. code-block:: javascript

    // within the App component
    async function updateTask(task: Task, newBody: string) {
        const url: string = `${API_HOST}/tasks/${task._id}`;
        const updated = Object.assign({}, task, {body: newBody});
        const response = await axios.put(url, updated);
        const newTask = response.data;

        setTasks(
            tasks
                .filter((task: Task) => task._id !== newTask._id)
                .concat(newTask)
                .sort((task1: Task, task2: Task) => {
                    let date1 = new Date(task1.creationDate);
                    let date2 = new Date(task2.creationDate);
                    return date1 > date2 ? 1 : -1;
                })
        );
    }

As we noted above, the function takes the ``task`` to be modified, as well as the new body that the task will have.
It creates a new object using the old task and the new body as a basis, then it sends a ``"PUT"`` request to the server with the updated version of the task.
When the server responds with the new version of the task, the old task is filtered out of the tasks array and the new task is added to the array.
Finally, we want to maintain the same order of tasks in the task list, so we leverage the ``creationDate`` of the task items to sort the array of tasks and put the updated task in its proper place.
In this particular case, we're placing older tasks at the front of the task list.
If we wanted to have newer tasks on top, we'd instead use ``return date1 > date2 ? -1 : 1``.

The return statement of the ``App`` component won't change too much.
Since all of the arguments being passed to ``TaskList`` have already been collected within ``taskListProps``, we can just add ``updateTask`` to that set of props.

.. code-block:: javascript

    // in the App component, right above the return statement.
    const taskListProps = { tasks, deleteTask, completeTask, updateTask };

Now that ``updateTask`` is built and passed down to ``TaskList``, we commit the change, then get to work on ``TaskList``.

.. code-block:: shell

    [update-task] $ git add src/App.tsx
    [udpate-task] $ git commit -m 'Added the updateTask function to the App component and passed it to TaskList'

So we have the server call, and that's great.
However, we have to decide on what the UI will involve.
Let's say we have a button that allows us to edit the task body.
How does that edit happen?
Does the div containing the task item turn into something else that takes user input?
Does the button click trigger a popup that darkens the rest of the window, waiting user input?
Do we use a classic JavaScript ``prompt``?
There's a number of ways to do this that influence how we build the UI, and none of them is canonically "the best".
Let's choose the first way and stick to it.

When we click the "edit" button, the div will turn into a ``textarea``, and the "edit" button will turn into a "save" button, which will save the changes made.
Let's establish a couple other rules to build this idea out a bit.

- When one task is being edited, no other tasks may be edited
- When "save" is clicked, the task item returns to being a static task item instead of a ``textarea``

One of the first things we can do is update the ``TaskList`` component to receive the ``updateTask`` function.
Then, we modify ``TaskList`` so that it manages the state of which items are currently being edited.
Once an item is identified as "being edited", we can use that identification to change the state of just that one item in the list and show that ``textarea``.

.. code-block:: javascript

    // in components/TaskList.tsx
    import React, { useState } from 'react';
    import { TaskItem } from './TaskItem';
    import { Task } from '../types';

    interface Props {
        tasks: Task[];
        deleteTask: (taskId: string) => void;
        completeTask: (task: Task) => void;
        updateTask: (task: Task, newBody: string) => void;
    }

    export const TaskList = ({ tasks, deleteTask, completeTask, updateTask }: Props) => {
        const [isEditing, setEditedTask] = useState('');
        const toBeEdited = (taskId: string) => {
            setEditedTask(taskId);
        };
        return <div>
            {tasks.map((task: Task) => {
                const itemProps = { 
                    key: task._id, task, deleteTask,
                    completeTask, toBeEdited, isEditing,
                    updateTask
                };
                return <TaskItem { ...itemProps } />;
            })}
        </div>;
    }

To start, we import ``useState`` from ``'react'`` at the top, since we'll need to make this ``TaskList`` stateful.
We also add ``updateTask`` to the ``Props`` interface and parameter list, so that ``TaskList`` can receive and use that function.

We set up ``TaskList`` to be stateful by calling ``useState`` with an empty string, returning the ID of the task being edited.
By default no task is being edited, so it stays set as an empty string.
When a task is selected to be edited, ``toBeEdited`` will be called and passed the taskId as an argument.
That taskId will be set to ``isEditing``, and then we'll know which task is actively being edited.

Each ``TaskItem`` will need to know if it's the one being edited, so we pass ``isEditing`` to each one.
Each one will also need to be able to be selected to be edited, so they all get ``toBeEdited`` as well.
Finally, when the update does happen, the new task body will need to be set, so each one will get ``updateTask``.

Let's commit these changes to ``TaskList`` and move on to the ``TaskItem``.

.. code-block:: shell

    [update-task] $ git add src/components/TaskList.tsx
    [udpate-task] $ git commit -m 'Functionality for maintaining a reference to the edited task has been added to the TaskList. All the necessary properties have also been passed down to the Task items'

Before we start making functional changes to ``TaskItem``, let's make sure that it can receive the new props being passed down.
It'll now be getting:

- ``isEditing``
- ``toBeEdited``
- ``updateTask``

With that in mind, let's update its ``Props`` interface and its parameter list.

.. code-block:: javascript

    // in components/TaskItem.tsx
    interface Props {
        isEditing: string;
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        toBeEdited: (taskId: string) => void;
        updateTask: (task: Task, newBody: string) => void;
    }

    export const TaskItem = ({
        task, deleteTask, completeTask,
        toBeEdited, updateTask 
    }: Props) => {

The ``TaskItem`` itself will become a bit more complex.
We'll have a variety of buttons living in the ``task-buttons`` section of each ``TaskItem``, along with a UI change depending on whether or not this individual ``TaskItem`` is being edited.
We'll also be adding complexity to the ``task-body`` section, having it change its appearance based on whether or not the ``TaskItem`` is being edited.

Let's break these out into their own components to make it easier to work with them individually.

.. code-block:: shell

    [update-task] $ touch src/components/{TaskButtons,TaskBody}.tsx

Within ``TaskButtons.tsx`` let's copy in the entire ``task-buttons`` div that current live in ``TaskItem.tsx``.
We need to make sure that the ``TaskButtons`` component can accept the right props to enable the ``deleteTask`` and ``completeTask`` operations it can already perform.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    import React from 'react';

    import { Task } from '../types';

    interface Props {
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
    }

    export const TaskButtons = ({ task, completeTask, deleteTask }: Props) => {
        return <div className="task-buttons">
            <button onClick={ () => completeTask(task) }>Complete</button>
            <button onClick={ () => deleteTask(task._id) }>Delete</button>
        </div>;
    }

Let's import this component into ``TaskItem.tsx``, taking its place where the ``task-buttons`` div used to be.

.. code-block:: javascript

    // in components/TaskItem.tsx
    import React from 'react';

    import { TaskButtons } from './TaskButtons';
    import { Task } from '../types';

    interface Props {
        isEditing: string;
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        toBeEdited: (taskId: string) => void;
        updateTask: (task: Task, newBody: string) => void;
    }

    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {  
        const buttonProps = { task, completeTask, deleteTask };
        return <div>
            <TaskButtons {...buttonProps} />
            <div className="task-body">
                {task.body}
            </div>
        </div>;
    }

Let's do the same with ``TaskBody.tsx``, noting that the only thing that the ``task-body`` div should care about at this moment is the body of the task.

.. code-block:: javascript

    // in components/TaskBody.tsx
    import React from 'react';

    interface Props {
        body: string;
    }

    export const TaskBody = ({ body }: Props) => {
        return <div className="task-body">{ body }</div>;
    }

And again, we want to import this component into ``TaskItem`` to replace what used to generate the body of the ``TaskItem`` component.

.. code-block:: javascript

    // in components/TaskItem.tsx
    import React from 'react';

    import { TaskBody } from './TaskBody';
    import { TaskButtons } from './TaskButtons';

    import { Task } from '../types';

    interface Props {
        isEditing: string;
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        toBeEdited: (taskId: string) => void;
        updateTask: (task: Task, newBody: string) => void;
    }

    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {  
        const buttonProps = { task, completeTask, deleteTask };
        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody body={ task.body } />
        </div>;
    }

There's been a lot of changes up to now, so let's commit these and continue working.

.. code-block:: shell

    [update-task] git add src/components/{TaskItem,TaskButtons,TaskBody}.tsx
    [update-task] git commit -m 'Broke the buttons and the body of TaskItem into its own components, and imported those new components into TaskItem'

We want to have a trigger to start off the editing process, so let's add a button to ``TaskButtons`` for editing.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    import React from 'react';

    import { Task } from '../types';

    interface Props {
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
    }

    export const TaskButtons = ({ task, completeTask, deleteTask }: Props) => {
        return <div className="task-buttons">
            <button>Edit</button>
            <button onClick={() => completeTask(task)}>Complete</button>
            <button onClick={() => deleteTask(task._id)}>Delete</button>
        </div>;
    }

Simple enough to start.
Now, what do we want to have happen when the button is clicked?
We want to set *this* task item as the one to be edited.
For that, we had the ``toBeEdited`` function, which takes a task ID as its argument.
Let's add ``toBeEdited`` to the set of ``buttonProps`` constructed in ``TaskItem``.

.. code-block:: javascript

    // within components/TaskItem.tsx
    const buttonProps = { task, completeTask, deleteTask, toBeEdited };

And add it to the props that ``TaskButtons`` will be receiving.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    interface Props {
        task: Task;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        toBeEdited: (taskId: string) => void;
    }

    export const TaskButtons = ({ task, completeTask, deleteTask, toBeEdited }: Props) => {
        return <div className="task-buttons">
            <button>Edit</button>
            <button onClick={() => completeTask(task)}>Complete</button>
            <button onClick={() => deleteTask(task._id)}>Delete</button>
        </div>;
    }

When the ``"Edit"`` button is clicked, we'll pass the task ID into ``toBeEdited``.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    export const TaskButtons = ({ task, completeTask, deleteTask, toBeEdited }: Props) => {
        return <div className="task-buttons">
            <button onClick={ () => toBeEdited(task._id) }>Edit</button>
            <button onClick={ () => completeTask(task) }>Complete</button>
            <button onClick={ () => deleteTask(task._id) }>Delete</button>
        </div>;
    }

That ID will be sent back up to the ``TaskList`` and set on ``isEditing`` to identify which ``TaskItem`` is being edited.
Come to think of it, we probably shouldn't see the ``"Edit"`` button if we're already editing task, right?
Let's use ``isEditing`` to put a more sensible ``"Save"`` button in its place.

First, let's pass ``isEditing`` on down through ``TaskItem`` into the ``TaskButtons`` component.

.. code-block:: javascript

    // within components/TaskItem.tsx
    const buttonProps = { task, isEditing, completeTask, deleteTask, toBeEdited };

Then, let's update the ``Props`` interface and parameter list of ``TaskButtons`` to receive ``isEditing``.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    interface Props {
        task: Task;
        isEditing: string;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        toBeEdited: (taskId: string) => void;
    }

    export const TaskButtons = ({
        task, isEditing,
        completeTask, deleteTask, toBeEdited
    }: Props) => {
        return <div className="task-buttons">
            <button onClick={ () => toBeEdited(task._id) }>Edit</button>
            <button onClick={ () => completeTask(task) }>Complete</button>
            <button onClick={ () => deleteTask(task._id) }>Delete</button>
        </div>;
    }

Finally, we can use a ternary operator to express the following: if this task is the one being edited, then instead of showing the "Edit" button, show a currently-non-functional "Save" button.
Otherwise, just show the "Edit" button as normal.

.. code-block:: javascript

    // in components/TaskButtons.tsx
    export const TaskButtons = ({
        task, isEditing,
        completeTask, deleteTask, toBeEdited
    }: Props) => {
        return <div className="task-buttons">
            {
                isEditing === task._id ?
                <button onClick={ () => {} }>Save</button>:
                <button onClick={ () => toBeEdited(task._id) }>Edit</button>
            }
            <button onClick={ () => completeTask(task) }>Complete</button>
            <button onClick={ () => deleteTask(task._id) }>Delete</button>
        </div>;
    }

When the pieces of my ternary operators get large I like to spread them onto separate lines, using the ``?`` and ``:`` characters kind of like punctuation marks to set my line breaks.

Let's make one more small change, then move on to the ``TaskBody``.
While that ``"Save"`` button doesn't need to update the database right away, we can at least use it to flip the editing status back to "no longer edited".
Instead of that empty arrow function within ``TaskButtons``, let's have the ``"Save"`` button just reset ``isEdited`` to an empty string.

.. code-block:: javascript 

    // in components/TaskButtons.tsx
    <button onClick={ () => toBeEdited('') }>Save</button> :

And now when the button is clicked, the Task will no longer be edited.
Let's commit these changes, then go on to the ``TaskBody``.

.. code-block:: shell

    [update-task] $ git add src/components/{TaskItem,TaskButtons}.tsx
    [update-task] $ git commit -m 'Allowed the TaskButtons component to know when its related task is being edited, and used a ternary operator to switch between an edit button and a nonfunctional save button'

Aside from the button-flip, we need to use the ``isEditing`` value to control the ``TaskBody``, displaying a ``textarea`` when in editing mode and the normal ``TaskBody`` when not.
Furthermore, when we're allowing the user to edit the task, it'd be great for them to be able to see what the body of the task *was* instead of being presented with a blank field.

Let's pass ``isEditing`` and the whole task into ``TaskBody`` from ``TaskItem``.

.. code-block:: javascript

    // in components/TaskItem.tsx
    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {
        const buttonProps = { task, isEditing, completeTask, deleteTask, toBeEdited };
        const bodyProps = { task, isEditing };

        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody {...bodyProps} />
        </div>;
    }

Following on down into the ``TaskBody``, we update its ``Props`` interface and parameter list to incorporate ``isEditing``, as well as switch out ``body: string`` for ``task: Task``.
Then, in a similar way in which we used the ``isEditing`` value to control whether or not the ``"Edit"`` or ``"Save"`` button was showing, we'll use ``isEditing`` to determine whether we're showing the regular task body or a ``textarea`` containing the task body as editable text.

.. code-block:: javascript

    // in components/TaskBody.tsx
    import React from 'react';
    import { Task } from '../types';

    interface Props {
        task: Task;
        isEditing: string;
    }

    export const TaskBody = ({ task, isEditing }: Props) => {
        return isEditing === task._id ?
            <textarea value={ task.body }/> :
            <div className="task-body">{ task.body }</div>;
    }

Let's commit this code, then actually allow the user to do something with the text field.

.. code-block:: shell

    [update-task] $ git add src/components/{TaskItem,TaskBody}.tsx
    [update-task] $ git commit -m 'The TaskItem has passed isEditing to the TaskBody and the TaskBody now can show either a text area or a div containing the tasks body.'

As we saw before with ``CreateTask``, in order for the user to actually be able to edit the task's body, it needs to be connected with an ``onChange`` event handler.
So what do we do?
What we want our code to express is that when the user changes the text in our ``textarea``, the value that will be stored on the task's body gets changed.

But this is tricky!
We only want the task's actual body to get changed when the user *chooses* to save the text.
We also don't want to fire off a request to the server for *every* keystroke the user makes.
What we'll need is an intermediary to maintain the state of what the task's body *might become* if the user decides to save the data.
Then, should the user decide to save it, that data is what gets sent to the server.

A good place to put such an intermediary is in the ``TaskItem`` component.
The ``TaskList`` is too high up in the hierarchy.
It shouldn't have to care about the contents of any individual task.

.. code-block:: javascript

    // in components/TaskItem.tsx
    // at the top
    import React, { useState } from 'react';

    // in the component
    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {
        const [ bodyText, setBodyText ] = useState(task.body);

        const buttonProps = { task, isEditing, completeTask, deleteTask, toBeEdited };
        const bodyProps = { task, isEditing };

        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody {...bodyProps} />
        </div>;
    }

``bodyText`` will be the intermediary state between the real and aspirational task body.
We'll pass this down into ``TaskBody`` and use the value to set what's in the ``textarea`` and the static task body.
We'll also pass down ``setBodyText``, as this will be the function called whenever the textarea receives a ``ChangeEvent``.

.. code-block:: javascript

    // in the TaskItem component
    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {
        const [ bodyText, setBodyText ] = useState(task.body);

        const buttonProps = { task, isEditing, completeTask, deleteTask, toBeEdited };
        const bodyProps = { task, isEditing, bodyText, setBodyText };

        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody {...bodyProps} />
        </div>;
    }

Of course, now that we've added to the props that ``TaskBody`` will receive, we need to update its interface and parameter list.

.. code-block:: javascript

    // in components/TaskBody.tsx
    import React, { Dispatch, SetStateAction } from 'react';
    import { Task } from '../types';

    interface Props {
        task: Task;
        isEditing: string;
        bodyText: string;
        setBodyText: Dispatch<SetStateAction<string>>
    }

    export const TaskBody = ({ task, isEditing, bodyText, setBodyText }: Props) => {
        return isEditing === task._id ?
            <textarea value={ task.body } /> :
            <div className="task-body">{ task.body }</div>;
    }

Note ``setBodyText``'s object type.
Those functions we've been getting back from ``useState`` are Dispatch type functions, used internally in ``React``.
Dispatch functions make something happen, and this particular type of Dispatch function fires off a ``SetStateAction``, whose job it is to set the state of a component.
The thing that'll be used to set that state is, in this particular case, a string.
If we had initialized ``useState`` with a number it'd be ``Dispatch<SetStateAction<number>>``.

Let's have this ``textarea`` manage the ``onChange`` event like we had planned above.

.. code-block:: javascript

    // in components/TaskBody.tsx
    // at the top
    import React, { Dispatch, SetStateAction, ChangeEvent } from 'react';

    // in the component
    export const TaskBody = ({ task, isEditing, bodyText, setBodyText }: Props) => {
        const updateText = (event: ChangeEvent<HTMLTextAreaElement>) => {
            setBodyText(event.target.value);
        }

        return isEditing === task._id ?
            <textarea
                value={ bodyText }
                onChange={ updateText }
            /> :
            <div className="task-body">{ task.body }</div>;
    }

Now, when the user first loads the ``TaskItem``, the current body will show.
As they type within the ``textarea``, ``Change`` events will fire.
``updateText`` will be listening for those ``Change`` events and respond by harvesting the current text within the ``textarea`` and updating the state of ``bodyText`` with it.
Note that while I could've just written ``updateText`` as a nameless arrow function within the ``onChange`` line, it was getting a little long so I made it its own function.
I also swapped out only one instance of ``task.body`` with ``bodyText``.
The ``task-body`` div should only ever show the currently stored value of the task body as it is represented on the server.
The ``textarea`` is fine to show what the edit could become via ``bodyText``.

These were big changes, so let's commit our progress thus far.
What's left now is to connect the call to the server and transmit our updated data.

.. code-block:: shell

    [update-task] $ git add src/components/{TaskItem,TaskBody}.tsx
    [update-task] $ git commit -m 'TaskItem is now stateful, storing the updated text for the task body as it has been typed within TaskBody.'

Thus far we've managed to create an edit button that signals which task is to be edited, create a text field that shows up when a task is being edited, and create a stateful object that maintains the state of the edited task.
Now we want to be able to take that data and send it server-side.
This is the type of operation that should only happen when the user chooses to save.

We want to write a expression that says "now that the user has chosen to save, we want to take whatever text is in the ``textarea`` and send it to the server.
After the server has received that change, we want to revert the ``textarea`` back to being a regular task body."
We can do this within ``TaskItem``.

.. code-block:: javascript

    // in components/TaskItem.tsx
    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {
        const [ bodyText, setBodyText ] = useState('');

        const saveBody = () => {
            updateTask(task, bodyText);
            toBeEdited('');
        }

        const buttonProps = { task, isEditing, completeTask, deleteTask, toBeEdited };
        const bodyProps = { task, isEditing, bodyText, setBodyText };

        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody {...bodyProps} />
        </div>;
    }

When the ``saveBody`` function is called, the new task body will be sent to the server, and the ID to be edited is reset, reverting the ``textarea`` within ``TaskBody`` back to a regular div containing some text.
We need this to be called when the ``"Save"`` button is clicked, so let's pass this function down through the props to ``TaskButtons``.

.. code-block:: javascript

    // in components/TaskItem.tsx
    export const TaskItem = ({
        task, deleteTask, completeTask,
        isEditing, toBeEdited, updateTask
    }: Props) => {
        const [ bodyText, setBodyText ] = useState('');

        const saveBody = () => {
            updateTask(task, bodyText);
            toBeEdited('');
        }

        const buttonProps = { 
            task, isEditing, completeTask,
            deleteTask, toBeEdited, saveBody
        };
        const bodyProps = { task, isEditing, bodyText, setBodyText };

        return <div>
            <TaskButtons {...buttonProps} />
            <TaskBody {...bodyProps} />
        </div>;
    }

Let's commit this, then continue on down to ``TaskButtons``.

.. code-block:: shell

    [update-task] $ git add src/components/TaskItem.tsx
    [update-task] $ git commit -m 'Created the function that enables the saving of an updated task body'

``TaskButtons`` is receiving a new prop, so its ``Props`` interface and parameter list need an update.

.. code-block:: javascript

    // in components/TaskBody.tsx
    interface Props {
        task: Task;
        isEditing: string;
        completeTask: (task: Task) => void;
        deleteTask: (taskId: string) => void;
        saveBody: () => void;
        toBeEdited: (taskId: string) => void;
    }

    export const TaskButtons = ({
        task, isEditing, completeTask,
        deleteTask, saveBody, toBeEdited
    }: Props) => {

This function will get called whenever the ``"Save"`` button gets clicked, so let's add it to the ``onClick`` handler.

.. code-block:: javascript

    // in components/TaskBody.tsx

    export const TaskButtons = ({
        task, isEditing, completeTask,
        deleteTask, saveBody, toBeEdited
    }: Props) => {
        return <div className="task-buttons">
            {
                isEditing === task._id ?
                    <button onClick={() => saveBody()}>Save</button> :
                    <button onClick={() => toBeEdited(task._id)}>Edit</button>
            }
            <button onClick={() => completeTask(task)}>Complete</button>
            <button onClick={() => deleteTask(task._id)}>Delete</button>
        </div>;
    }

And that's it!
We can now update and save a task on the client.
It was a long road to get here, but the functionality of the app is effectively complete!
Make sure to fire up both the server and client, and try it out in the browser.
Then commit, merge, and let's talk about adding CSS.

.. code-block:: shell

    [update-task] $ git add src/components/TaskBody.tsx
    [update-task] $ git commit -m 'Updated the TaskBody to include the saveBody functionality.'
    [update-task] $ git checkout master
    [master] $ git merge update-task -m 'Tasks can now be updated from the client side by clicking the "Save" button on an edited task.'
