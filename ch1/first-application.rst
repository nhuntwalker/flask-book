=============
Hello, World!
=============

.. important::

    Do a single file application that returns a text response of "Hello, World."

It'll end up coming more-or-less directly from the official `Flask documentation <http://flask.pocoo.org/>`_.

.. code-block:: python

    from flask import Flask
    app = Flask(__name__)

    @app.route("/")
    def home() -> str:
        """Home route for our Flask application."""
        return "Hello World!"
