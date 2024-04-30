Introduction to Flask
=====================

In this section, we will get a brief introduction into Flask, the Python web framework,
including how to set up a REST API with multiple routes (URLs). After going through this
module, students should be able to:

* Install the Python Flask library and import it into a Python program.
* Define and implement various "routes" or API endpoints in a Flask Python program.
* Run a local Flask development server.
* Use curl to test routes defined in their Flask program when the local Flask development
  server is running.
* **Design Principles**: Additionally, we will see how using Flask contributes to 
  the *modularity*, *abstraction*, and *generalization* of software. 


Flask is a Python library and framework for building web servers. Some of the
defining characteristics of Flask make it a good fit for this course:

* Flask is small and lightweight - relatively easy to use and get setup initially
* Flask is robust - a great fit for REST APIs and **microservices**
* Flask is performant - when used correctly, it can handle the traffic of sites
  with 100Ks of users


What is a Microservice?
-----------------------

Microservices - also known as the microservice architecture - is an
architectural style that structures an application as a collection of services
that are:

* Highly maintainable and testable
* Loosely coupled
* Independently deployable
* Organized around business capabilities

The microservice architecture enables the continuous delivery/deployment of
large, complex applications. It also enables an organization to evolve its
technology stack. Many heavily-used, well-known sites use microservices
including Netflix, Amazon, and eBay.

There is a great article on DevTeam.Space
`about microservices <https://www.devteam.space/blog/microservice-architecture-examples-and-diagram/>`_.


Setup and Installation
----------------------

The Flask library is not part of the Python standard library but can be
installed with standard tools like ``pip3``. In addition to making Flask available to
import into a Python program, it will also expose some new command line tools. On
your Jetstream VM, perform the following:

.. code-block:: console

   [user-vm]$ pip3 install --user flask
   ...
   Successfully installed flask-3.0.2

   [user-vm]$ flask --help
   Usage: flask [OPTIONS] COMMAND [ARGS]...

     A general utility script for Flask applications.

     An application to load must be given with the '--app' option, 'FLASK_APP'
     environment variable, or with a 'wsgi.py' or 'app.py' file in the current
     directory.

   Options:
     -e, --env-file FILE   Load environment variables from this file. python-
                           dotenv must be installed.
     -A, --app IMPORT      The Flask application or factory function to load, in
                           the form 'module:name'. Module can be a dotted import
                           or file path. Name is not required if it is 'app',
                           'application', 'create_app', or 'make_app', and can be
                           'name(args)' to pass arguments.
     --debug / --no-debug  Set debug mode.
     --version             Show the Flask version.
     --help                Show this message and exit.

   Commands:
     routes  Show the routes for the app.
     run     Run a development server.
     shell   Run a shell in the app context.


.. tip::

   If you aren't already using a virtual environment to help manage your Python
   libraries, now is a `good time to start <https://docs.python.org/3/library/venv.html>`_!


A Hello World Flask App
-----------------------

In a new directory on the class server, create a file called ``app.py`` and open
it for editing. Enter the following lines of code:

.. code-block:: python3
   :linenos:

   from flask import Flask

   app = Flask(__name__)

   # the next statement should usually appear at the bottom of a flask app
   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')

On the first line, we are importing the Flask class.

On the third line, we create an instance of the Flask class (called ``app``).
This so-called "Flask application" object holds the primary configuration and
behaviors of the web server.

Finally, the ``app.run()`` method launches the development server. The
``debug=True`` option tells Flask to print verbose debug statements while the
server is running. The ``host=0.0.0.0`` option instructs the server to listen
on all network interfaces; basically this means you can reach the server from
inside and outside the host VM.


Run the Flask App
-----------------

There are a few options when starting the Flask app. For now, we recommend you
start your Flask application using the ``flask run`` command, specifying the name 
of the Python file (in our case ``app.py``) using the ``--app`` option, and 
running in debug mode using the ``--debug`` flag.

.. code-block:: console

   [user-vm]$ flask --app app --debug run
   * Serving Flask app 'app'
   * Debug mode: on
   WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
   * Running on http://127.0.0.1:5000
   Press CTRL+C to quit
   * Restarting with stat
   * Debugger is active!
   * Debugger PIN: 268-620-354

That's it! We now have a server up and running. Some notes on what is happening:

* Note that the program took over our shell; we could put it in the background,
  but for now we want to leave it in the foreground. (Multiple PIDs are started
  for the Flask app when started in daemon mode; to get them, find all processes
  listening on the port 5000 socket with ``lsof -i:5000``).
* If we make changes to our Flask app while the server is running in development
  mode, the server will detect those changes automatically and "reload"; you will
  see a log to the effect of ``Detected change in <file>``.
* We can stop the program with ``Ctrl+C`` just like any other (Python) program.
* If we stop our Flask programs, the server will no longer be listening and our
  requests will fail.

.. note::

  The order of the arguments and command is important. Be sure the ``--app``
  and ``--debug`` parameters appear **before** ``run``.


Next we can try to talk to the server using ``curl``. Note this line:

.. code-block:: console

     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

That tells us our server is listening on the ``localhost`` - ``127.0.0.1``, and
on the default Flask port, port ``5000``.

Ports Basics
~~~~~~~~~~~~

Ports are a concept from networking that allows multiple services or programs to
be running at the same time, listening for messages over the internet, on the
same computer.

* For us, ports will always be associated with a specific IP address. In
  general, we specify a port by combining it with an IP separated by a colon (``:``)
  character. For example, ``129.114.97.16:5000``.
* One and only one program can be listening on a given port at a time.
* Some ports are designated for specific activities; Port 80 is reserved for
  HTTP, port 443 for HTTPS (encrypted HTTP), but other ports can be used for
  HTTP/HTTPS traffic.

.. note::

   Only one application can be associated with a given port. If you try to 
   run a second Flask application on the same default port (5000) on the 
   same machine, you will hit errors. You can specify the port you want
   Flask to listen on using the ``-p`` (or ``--port``) option to the 
   ``flask run`` command; e.g., 
   ``flask --app app --debug run -p 5001``
   

curl Basics
~~~~~~~~~~~

You can think of ``curl`` as a command-line version of a web browser: it is just
an HTTP client.

* The basic syntax is ``curl <some_base_url>:<some_port>/<some_url_path>``.
  This will make a ``GET``
  request to the URL and port print the message response.
* Curl will default to using port 80 for HTTP and port 443 for HTTPS.
* You can specify the HTTP verb to use with the ``-X`` flag; e.g.,
  ``curl -X GET <some_url>`` (though ``-X GET`` is redundant because that is the
  default verb).
* You can set "verbose mode" with the ``-v`` flag, which will then show
  additional information such as the headers passed back and forth (more on this
  later).

Try the following, for example: 

.. code-block:: console

   [user-vm]$ curl https://api.github.com

Make a Request
--------------

Because the terminal window running your Flask app is currently locked to that
process, the simplest thing to do is open up a new terminal and SSH into the
class server again.

To make a request to your Flask app, type the following in the new terminal:

.. code-block:: console

   [user-vm]$ curl 127.0.0.1:5000
   - or -
   [user-vm]$ curl localhost:5000


You should see something like the following response:

.. code-block:: html

   <!doctype html>
   <html lang=en>
   <title>404 Not Found</title>
   <h1>Not Found</h1>
   <p>The requested URL was not found on the server. If you entered the URL manually please
   check your spelling and try again.</p>


Our server is sending us HTML! It's sending a 404 that it could not find the
resource we requested. Although it appears to be an error (and technically it
is), this is evidence that the Flask server is running successfully. It's time
to add some routes.


Routes in Flask
---------------

In a Flask app, you define the URLs in your application using the ``@app.route``
decorator. Specifications of the ``@app.route`` decorator include:

* Must be placed on the line before the declaration of a Python function.
* Requires a string argument which is the path of the URL (not including the base
  URL)
* Takes an argument ``methods`` which should be a list of strings containing the
  names of valid HTTP methods (e.g. ``GET``, ``POST``, ``PUT``, ``DELETE``)

When the URL + HTTP method combination is requested, Flask will call the
decorated function.


Tangent: What is a Python Decorator?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A decorator is a function that takes another function as an input and extends
its behavior in some way. The decorator function itself must return a function
which includes a call to the original function plus the extended behavior. The
typical structure of a decorator is as follows:

.. code-block:: python3
   :linenos:

   def my_decorator(some_func):

       def func_to_return():

           # extend the behavior of some_func by doing some processing
           # before it is called (optional)
           do_something_before()

           # call the original function
           some_func(*args, **kwargs)

           # extend the behavior of some_func by doing some processing
           # after it is called (optional)
           do_something_after()

       return func_to_return

As an example, consider this test program:

.. code-block:: python3
   :linenos:

   def print_decorator(f):
       def func_to_return(*args, **kwargs):
           print(f'args: {args}; kwargs: {kwargs}')
           val = f(*args, **kwargs)
           print(f'return: {val}')
           return val
       return func_to_return

   @print_decorator
   def foo(a):
       return a+1

   result = foo(2)
   print(f'Got the result: {result}')

Our ``@print_decorator`` decorator gets executed automatically when we call ``foo(2)``.
Without the decorator, the final output would be:

.. code-block:: text

   Got the result: 3

By using the decorator, however, the final output is instead:

.. code-block:: text

   args: (2,); kwargs: {}
   return: 3
   Got the result: 3

Define the Hello World Route
----------------------------

The original Flask app we wrote above (in ``app.py``) did not define any routes.
Let's define a "hello world" route for the base URL. Meaning if someone were to
curl against the base URL (``/``) of our server, we would want to return the
message "Hello, world!". To do so, add the following lines to your ``app.py``
script:

.. code-block:: python3
   :linenos:
   :emphasize-lines: 5-7

   from flask import Flask

   app = Flask(__name__)

   @app.route('/', methods=['GET'])
   def hello_world():
       return 'Hello, world!\n'

   # the next statement should usually appear at the bottom of a flask app
   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')

The ``@app.route`` decorator on line 5 is expecting ``GET`` requests at the base
URL ``/``. When it receives such a request, it will execute the ``hello_world()``
function below it.

In your active SSH terminal, execute the curl command again (you may need to
restart the Flask app); you should see:

.. code-block:: console

   [user-vm]$ curl localhost:5000/
   Hello, world!

Routes with URL Parameters
--------------------------

Flask makes it easy to create routes (or URLs) with variables in the URL. The
variable name simply must appear in angled brackets (``<>``) within the
``@app.route()`` decorator statement. Then, specify the variable as a parameter 
to the actual function.

For example, the following would grant the
function below it access to a variable called ``year``:

.. code-block:: python3

   @app.route('/<year>', methods=[...])
   def f(year):
       # function implementation...


In the next example, we extend our ``app.py`` Flask app by adding a route
with a variable (``<name>``):

.. code-block:: python3
   :linenos:
   :emphasize-lines: 9-11

   from flask import Flask

   app = Flask(__name__)

   @app.route('/', methods=['GET'])
   def hello_world():
       return 'Hello, world!\n'

   @app.route('/<name>', methods=['GET'])
   def hello_name(name):
       return f'Hello, {name}!\n'

   # the next statement should usually appear at the bottom of a flask app
   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')

Now, the Flask app supports multiple routes with different functionalities:

.. code-block:: console

   [user-vm]$ curl localhost:5000/
   Hello, world!
   [user-vm]$ curl localhost:5000/joe
   Hello, joe!
   [user-vm]$ curl localhost:5000/jane
   Hello, jane!


EXERCISE
~~~~~~~~

Let's use the sample Meteorite Landing data (`see here <https://raw.githubusercontent.com/TACC/coe-332-sp24/main/docs/unit02/sample-data/Meteorite_Landings.json>`_)
to define some more interesting routes. We will create a route that allows a user
to download the entire dataset over HTTP. Consider the following:

* What should the name of our function be?
* What URL path should it respond to?
* What HTTP verb(s) should it handle?

Once those questions are answered, we'll need to actually implement the new route function.
What will we need to do to implement the function? The implementation will require two
steps:

1) Read the data into Python from the JSON file. (What Python library will you use for this step?)
2) Return the result of step 1)

Once implemented, test the function using ``curl``.

Next, write one more route to access the information of a specific meteorite.
In REST API parlance, assume the whole data set is a "collection", and the data
from one meteorite is an "item" of that collection.


Additional Resources
--------------------

* `Flask Docs <https://flask.palletsprojects.com/en/3.0.x/>`_
