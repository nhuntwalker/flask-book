==================================
Installing Python, Git, and NodeJS
==================================

**Objectives**

Ensure there are working installations of

- Python 3.7+
- Git 
- NodeJS

Make sure to have Python 3.7+, Git (I'm running version 2.17.2), and NodeJS (I'm running version 11.0.0, but 10.15.3+ should be fine) installed and running on your machine.
We'll need other software down the line, but this is enough to get started.
Whether you have them or not, this section will run through some instructions on how to obtain and install them.

Note: while the details here are specific to those using Mac OSX or Linux (Debian/Ubuntu), they all have installation directions for Windows and Red Hat users available on their respective websites.
For the OSX users, I handle all software installs that I can with Homebrew [#f1]_.

Installing Python on Mac OSX
----------------------------

While Mac OSX will come with Python already installed, as of this writing the default version is some variant of Python 2.7.
Because Python 2.7's long term support ends at the start of 2020, and because Python 3.7 is better, we're going to make sure we have Python 3.7+ installed and available.
If you already have Python 3.7+ installed, feel free to skip to the next section on installing NodeJS.

With Homebrew, Python 3.7+ installation goes as

.. code-block:: shell

    $ brew install python3

This will install the latest version of Python 3 for you, as well as Python's package manager, ``pip``.
If you're partial to using ``pipenv`` instead of ``pip`` be my guest, however this book will use ``pip`` moving forward.

Installing Python on Debian/Ubuntu
----------------------------------

Assuming you're using Ubuntu version 18.04.2 LTS, you can download and install Python 3.7 in the following way using ``apt``

.. code-block:: shell

    $ sudo apt update
    $ sudo apt upgrade
    $ sudo apt install software-properties-common
    $ sudo add-apt-repository ppa:deadsnakes/ppa
    $ sudo apt install python3.7

Unlike the OSX installation, ``pip`` doesn't come with the installation of Python 3.7 and must be installed separately.
That can be done with the following command

.. code-block:: shell

    $ sudo apt install python3-pip

Installing Git on OSX
---------------------

OSX should already come with ``git`` installed.
You can check if you have it with

.. code-block:: shell

    $ which git

Your shell should respond with something along the lines of ``/usr/bin/git``.
However, if your machine doesn't have ``git`` installed and available, you can get it with Homebrew.

.. code-block:: shell

    $ brew install git

Installing Git on Debian/Ubuntu
-------------------------------

It's far more likely that a Linux system won't have ``git`` available, especially if you're provisioning servers yourself.
Thankfully, much like with the installation of Python, ``apt`` can take care of installation for you.

.. code-block:: shell

    $ sudo apt install git

Installing NodeJS on OSX
------------------------

With Homebrew, you can install NodeJS just as you had installed Python and Git.

.. code-block:: shell

    $ brew install node

This will install not just the latest versin of NodeJS, but also the Node Package Manager ``npm``.
This becomes very useful when we start building our front-ends.
We're specifically going to want an extension called ``npx`` for using the latest versions of ``create-react-app``, so let's install that globally.

.. code-block:: shell

    $ npm install -g npx

Installing NodeJS on Debian/Ubuntu
----------------------------------

Much like with installing Git, installing NodeJS and ``npm`` involve a one-line call to ``apt``

.. code-block:: shell

    $ sudo apt install nodejs npm

Note how NodeJS and npm are installed separately.
Now that you have ``npm``, use it to install ``npx`` globally.
You may need to use ``sudo`` here.

.. code-block:: shell

    $ sudo npm install -g npx

Python Virtual Environments
---------------------------

Although you've just downloaded and installed Python 3.7+, it isn't set to be your default Python, which is OK.
We don't want to be installing packages globally anyway, unless we have a very good reason to do so.

Python 3.7 allows us to create **virtual environments** on demand.
**Virtual environments** are little universes within which the version of Python is what we set it to be, and any installs are local to that environment, generally leaving the global Python package library alone.

To start a virtual environment with Python 3.7, we navigate to whatever directory we want to work in, then use use the ``venv`` module to create and name the new environment.

.. code-block:: shell

    $ python3.7 -m venv ENV

I tend to call all of my virtual environments ``ENV``, as that stands out when I'm listing the contents of a directory.
It's also ignored by default by `GitHub's Python .gitignore <https://github.com/github/gitignore/blob/master/Python.gitignore>`_, allowing us to not even have to think about it when we're starting a new repository.

Once the virtual environment is created, we can activate it by sourcing the new environment.

.. code-block:: shell

    $ source ENV/bin/activate
    (ENV) $

This will add our new, isolated version of Python to our ``PATH``.
This comes with a few benefits:

- Any package installs will be within the ``ENV/lib/python3.7/site-packages`` directory
- Any environment variables that we want/need to declare can be added to ``ENV/bin/activate`` without polluting our global environment and its variables
- If, for whatever reason, we decide that it's time to tear down the environment, we do so with ``rm -rf ENV`` without consequence to the global system
