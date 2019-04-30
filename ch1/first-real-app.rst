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
Let's commit and merge our code before we move forward.

.. code-block:: shell

    (ENV) [first-route] $ git add app.py tasks.py
    (ENV) [first-route] $ git commit -m 'Completed the first route with some sample data. Will remove sample data once the database is created.'
    (ENV) [first-route] $ git checkout master
    (ENV) [master] $ git merge first-route

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
        return (<div className="App></div>);
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
        return (<div className="App></div>);
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
We can do this within the ``<div className="App"></div>`` container div.

.. code-block:: javascript

    const App: FunctionComponent = () => {
        const [ tasks, setTasks ] = useState<Array<Task>>([]);
        return (
            <div className="App">
                { tasks.map((task: Task) => <div>{ task.body }</div>) }
            </div>
        );
    }

Here we're taking the body of every task and using it as content for the ``<div>`` that contains it.
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
                { tasks.map((task: Task) => <div>{ task.body }</div>) }
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
        { tasks.map((task: Task) => <div>{ task.body }</div>) }
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
                {tasks.map((task: Task) => <div>{task.body}</div>)}
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
    [list-tasks] $ git merge list-tasks

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
Time to commit.

.. code-block:: shell

    (ENV) [add-database] $ git add app.py 
    (ENV) [add-database] $ git commit -m 'Connected the mongodb client and updated the get_tasks function to serve data from the database instead of directly from file.'

Submitting New Data from the Client
-----------------------------------

