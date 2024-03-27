Advanced Flask
==============

We continue using Flask in this module with a look at more complex endpoints and data retrieval
functions for our REST API. After going through this module, students should be able to:

* Identify valid and invalid Flask route return types
* Convert unsupported types (e.g. ``int``) to valid Flask route return types
* Extract Content-Type and other headers from Flask route responses
* Add query parameters to GET requests, and extract their values inside Flask routes
* Deal with errors from user-supplied input to an API and handle Python exceptions
* Handle multiple request methods to support CRUD operations

.. note::

   We will continue to work on the the individual student VMs. Like last time, it will be helpful for you to
   have two SSH terminals open to your VM at the same time so you can run your Flask application in
   one terminal and test it in the other.


Defining the URLs of Our API
----------------------------

One of our first goals for our API will be to provide an interface to a dataset. Since
the URLs in a REST API are defined by the "nouns" or collections of the
application domain, we can use a noun that represents our data.

For example, suppose we have the following dataset that represents the number of
students earning an undergraduate degree for a given year:

.. code-block:: python3

   def get_data():
       return [ {'id': 0, 'year': 1990, 'degrees': 5818},
                {'id': 1, 'year': 1991, 'degrees': 5725},
                {'id': 2, 'year': 1992, 'degrees': 6005},
                {'id': 3, 'year': 1993, 'degrees': 6123},
                {'id': 4, 'year': 1994, 'degrees': 6096} ]


In this case, one collection described by the data is "degrees". So, let's
define a route, ``/degrees``, that by default returns all of the data points.

EXERCISE 1
~~~~~~~~~~

Create a new file, ``degrees_api.py`` to hold a Flask application then do the
following:

1) Import the Flask class and instantiate a Flask application
   object.
2) Add code so that the Flask server is started when this file is executed
   directly by the Python interpreter.
3) Copy the ``get_data()`` method above into the application
   script.
4) Add a route (``/degrees``) which responds to the HTTP ``GET`` request and
   returns the complete list of data returned by ``get_data()``. 

In a separate Terminal use ``curl`` to test out your new route. Does it work as
expected?

.. tip::

   Refer back to the `Intro to Flask material <intro_to_flask.html>`_ if
   you need help remembering the boiler-plate code.


EXERCISE 2
~~~~~~~~~~
Back inside the ``degrees_api.py`` file, let's add a second route, ``/degrees/<id>`` that returns the 
data associated with a single dictionary. There are often design questions one should consider when writing 
new code. In this case, we have:

  * What method(s) should it accept? 
  * What type will the incoming ``id`` field be from the user? 
  * How will you find the corresponding dictionary? 
  * What should happen if the user enters an ``id`` that doesn't exist?


Discussion
^^^^^^^^^^
By default, Flask uses String for the types of path variables. If we use a route declaration like this,

.. code-block:: python3
   
   @app.route('/degrees/<id>', methods=['GET'])
   def degrees_for_id(id):
       # implementation...


Then GET any request with a URL path that starts with ``/degrees/`` and ends with any string will match. That is,

  * ``/degress/0`` --> ``id`` holds the value ``"0"`` as a Python String.
  * ``/degrees/A`` --> ``id`` holds the value ``"A"`` as a Python String.
  * ``/degrees/one`` --> ``id`` holds the value ``"one"`` as a Python String.

will all match the ``degrees_for_id`` route and the variable, ``id`` will hold a ``str`` value. In this case,
we'll have to deal with the ``str`` type in our function, converting it to ``int``, etc.  


Typed URL Parameters
---------------------

We can specify the types of the URL parameters we are expecting using the syntax ``<type:variable_name>``. 
For example, we could change our ``degrees_for_id`` route declaration as follows, to indicate we required the ``id``
variable to be an integer:

.. code-block:: python3
   
   @app.route('/degrees/<int:id>', methods=['GET'])
   def degrees_for_id(id):
       # implementation...

With the above definition, a request like ``GET /degrees/A`` will no longer match our ``degrees_for_id`` route
while a request like ``GET /degrees/2`` will ``call degrees_for_id`` with an integer type for the ``id``
variable. 

Here is a summary of the types supported in Flask; see the `docs <https://flask.palletsprojects.com/en/3.0.x/quickstart/#routing>`_
for more details. 

.. list-table:: Type Support in Flask URL Path Parameters
   :widths: 10 25
   :header-rows: 1

   * - Type 
     - Support
   * - string
     - (default) accepts any text without a slash
   * - int
     - accepts positive integers
   * - float 
     - accepts positive floating point values
   * - path
     - like string but also accepts slashes
   * - uuid 
     - accepts UUID strings

.. warning::

   The numeric types, ``int`` and ``float`` do **not** accept negative values!


EXERCISE 3
~~~~~~~~~~
Modify your ``degrees_for_id`` route to specify an integer path parameter. 


Responses in Flask
------------------

Suppose we wanted to add a third route that just returns a single value, the number of degrees associated with a 
a particular dictionary. We might proceed as follows:

  * For the URL path, use ``/degrees/<int:id>/degrees``
  * Iterate through the list looking for the dictionary with the same id as the input. 
  * If we find a dictionary, ``d``, with the same id, return ``d['degrees']``.

Let's try that and see what happens.


EXERCISE 4
~~~~~~~~~~
Implement a new route for the ``/degrees/<int:id>/degrees`` endpoint. Does it work as you expect? 


If you tried to return the integer object, ``d['degrees']`` directly in your route function
definition, you got an error when you tried to request it with curl. A long stack trace is returned, 
but at the end you will see:

.. code-block:: console

   TypeError: The view function did not return a valid response. The return type must
   be a string, dict, list, tuple with headers or status, Response instance, or WSGI
   callable, but it was a int.


Flask allows you four options for creating responses:

1) Return a string (``str``) object
2) Return a dictionary (``dict``) or list ``list`` object
3) Return a tuple (``tuple``) object in particular form -- we'll return to this later. 
4) Return a ``flask.Response`` object

Some notes:

* Option 1 is good for text or html such as when returning a web page or text
  file.
* Option 2 is good for returning rich information in JSON format.
* Option 3 is good for returning additional information including headers and status code. 
* Option 4 gives you the most flexibility, as it allows you to customize the
  headers and other aspects of the response.

For our REST API, we will want to return JSON-formatted data. Flask will handle all of this for us,
so long as we return a list or dictionary. 

.. tip::

   Refer back to the `Working with JSON material <../unit02/json.html>`_ for a
   primer on the JSON format and relevant JSON-handling methods.



Returning JSON (and Other Kinds of Data)
----------------------------------------

You probably are thinking at this point we can fix our solution to Exercise 4
by changing the return type. Instead of returning a raw integer, we can return a type that Flask recognized. 
What type should we return?


EXERCISE 5
~~~~~~~~~~

Update your code from Exercise 4 to return a Python type that Flask accepts.
Then, with your API server running in one window, open a Python interactive
session in another window and:

* Make a ``GET`` request to your ``/degrees`` URL and capture the response in a
  variable, say ``r``
* Verify that ``r.status_code`` is what you expect (what do you expect it to be?)
* Verify that ``r.content`` is what you expect.
* Use ``r.json()`` to decode the response and compare the type to that of ``r.content``.

Then, repeat the above with the ``/degrees/<id>/degrees`` endpoint. 


HTTP Content Type Headers
-------------------------

Requests and responses have ``headers`` which describe additional metadata about
them. Headers are ``key:value`` pairs (much like dictionary entries). The ``key``
is called the header name and the ``value`` is the header value.

There are many pre-defined headers for common metadata such as specifying the
size of the message (``Content-Length``), the domain the server is listening on
(``Host``), and the type of content included in the message (``Content-Type``).


We can use ``curl`` or the Python ``requests`` library to see all of the headers
returned on a response from our Flask server. Let's try it.

EXERCISE 6
~~~~~~~~~~

1) Use ``curl`` to make a GET request to your ``/degrees`` endpoint
   and pass the ``-v`` (for "verbose") option. This will show you additional information,
   including the headers. Note that with ``-v``, curl shows headers on both the request and
   the response. Request headers are lines that start with a ``>`` while response headers are
   lines that start with a ``<``.
2) Use ``curl`` again to make the same request, but this time pass the ``--head``
   option instead of the ``-v``; this will show you **only** the headers being
   returned in the response.
3) Inside a Python shell, use ``requests`` to make the same GET request to your ``/degrees``
   endpoint, and capture the result in a variable, ``r``. Inspect the ``r.headers`` attribute.
   What is the type of ``r.headers``?


.. code-block:: console

   [user-vm]$ curl localhost:5000/degrees -v
   * Trying 127.0.0.1:5000...
   * TCP_NODELAY set
   * Connected to localhost (127.0.0.1) port 5000 (#0)
   > GET /degrees HTTP/1.1
   > Host: localhost:5000
   > User-Agent: curl/7.68.0
   > Accept: */*
   > 
   * Mark bundle as not supporting multiuse
   < HTTP/1.1 200 OK
   < Server: Werkzeug/2.2.2 Python/3.8.10
   < Date: Sun, 12 Feb 2023 16:42:55 GMT
   < Content-Type: application/json
   < Content-Length: 303
   < Connection: close
   < 

.. code-block:: python3

   >>> import requests
   >>>
   >>> response = requests.get('http://127.0.0.1:5000/degrees')
   >>>
   >>> response.headers
   {'Server': 'Werkzeug/2.2.2 Python/3.8.10', 'Date': 'Sun, 12 Feb 2023 16:41:23 GMT',
   'Content-Type': 'application/json', 'Content-Length': '49', 'Connection': 'close'}

We see that we are sending a ``Content-Type`` of ``'application/json'``, which is what we want. That is how the
Python requests library is able to provide the ``r.json()`` function to automatically convert to a Python list or 
dictionary. 


Media Type (or Mime Type)
~~~~~~~~~~~~~~~~~~~~~~~~~

The allowed values for the ``Content-Type`` header are the defined
**media types** (formerly, **mime types**). The main thing you want to know
about media types are that they:

* Consist of a type and subtype
* The most common types are application, text, audio, image, and multipart
* The most common values (type and subtype) are application/json,
  application/xml, text/html, audio/mpeg, image/png, and multipart/form-data


Query Parameters
----------------

The HTTP specification allows for parameters to be added to the URL in form of
``key=value`` pairs. Query parameters come after a ``?`` character and are
separated by ``&`` characters; for example, the following request to a hypothetical API:

.. code-block:: console

      GET https://api.example.com/degrees?limit=3&offset=2

passes two query parameters: ``limit=3`` and ``offset=2``. Note that the URL path in
the example above is still ``/degrees``; that is, the ``?`` character terminates the URL
path, and any characters that follow create the query parameter set for the request.

In REST architectures, query parameters are often used to allow clients to
provide additional, optional arguments to the request.

Common uses of query parameters in RESTful APIs include:

* Pagination: specifying a specific page of results from a collection
* Search terms: filtering the objects within a collection by additional search
  attributes
* Other parameters that might apply to most if not all collections such as an
  ordering attribute (``ascending`` vs ``descending``)


Extracting Query Parameters in Flask
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Flask makes the query parameters available on the ``request.args`` object, which
is a "dictionary-like" object. To work with the query parameters supplied on a
request, you must import the Flask ``request`` object, and use the ``args.get`` method to
extract the passed query parameter into a variable.

.. note::

  The ``flask.request`` object is different from the Python ``requests`` library we used to
  make http requests. the ``flask.request`` object represents the incoming request that our
  flask application server has received from the client.

For example, consider the following line of Python code: 

.. code-block:: python3

   start = request.args.get('start')

In this case, the start variable will be the value of the start parameter if one is passed, or it 
will be None otherwise.

.. note::

   ``request.args.get()`` will always return a ``string``, regardless of the
   type of data being passed in.



EXERCISE 7
~~~~~~~~~~
Implement the ``start`` query parameter on your ``GET /degrees`` endpoint and check the behavior by
issuing some ``curl`` requests in another window, e.g.,  

.. code-block:: console

   [user-vm]$ curl http://api.example.com/degrees?start=1993


Let's use this idea to update our ``degrees_api`` to only return the years starting from the
``start`` query parameter year, if that parameter is provided.




Solution
~~~~~~~~~

To implement a ``start`` query parameter on the ``GET /degrees`` endpoint that only returns data
for years on or after the ``start`` year, we first might write something like the following:

.. code-block:: python3
   :linenos:

   @app.route('/degrees', methods=['GET'])
   def degrees():
       start = request.args.get('start')
       data = get_data()
       # iterate through data and check if years are >= start...

However, there are a couple of issues here:

  1. The user may not provide a ``start`` query parameter, in which case our ``start`` variable will be ``None``.
  2. If the user does provide a ``start`` query parameter, it will be a string type, which cannot be compared to 
     an integer year. 

Here is a first approach to fixing it: 

.. code-block:: python3
   :linenos:

   from flask import Flask, request

   @app.route('/degrees', methods=['GET'])
   def degrees():
      # provide a default value that is less than all the years and 
      start = int(request.args.get('start', 0))
      data = get_data()
      result = []
      for d in data:
         if d['year'] >= start:
               result.append(d)
      return result



Error Handling
--------------

However, there is one more problem with our solution above: What happens if the user enters a non-numeric
value for the ``start`` parameter? Try it and see what happens:

.. code-block:: console

   [user-vm]$ curl http://127.0.0.1:5000/degrees?start=abc


Yikes! If we try this we get a long traceback that ends like this:

.. code-block:: console

    . . . 
    File "/home/ubuntu/test/degrees_api.py", line 26, in degrees2
      start = int(request.args.get('start', 0))
    ValueError: invalid literal for int() with base 10: 'abc'


Checking User Input
~~~~~~~~~~~~~~~~~~~

If we get a request like this, in the best case, the user didn't understand what kind of data to put
in the ``start`` query parameter; in the worst case, they were intentionally trying to send our
program bad data to break it. We should always be very careful with user-supplied data and make sure
it contains the kind of data we expect.

So, what is it we expect from the ``start`` query parameter? At a minimum, it needs to be some kind
of integer data, because we are casting it to the ``int`` type. Therefore, at a minimum, we should
check if it is an integer.

We can use the Python ``isnumeric()`` method on a Python string to test whether a string
contains non-negative integer data.

Let's try some examples in the Python shell:

.. code-block:: python3

   >>> '123'.isnumeric()
   True
   >>> 'abc'.isnumeric()
   False
   >>> '1.23'.isnumeric()
   False
   >>> '-1'.isnumeric()
   False


Now, let's fix our route function; we can check if it is numeric before casting to an ``int``.
If it is not numeric, we can return an error message to the user.

.. code-block:: python3
   :linenos:

    @app.route('/degrees', methods=['GET'])
    def degrees():
        start = request.args.get('start', 0)
        if not start.isnumeric():
            return "Error: start must be an integer\n"
        start = int(start)
        data = get_data()
        result = []
        for d in data:
            if d['year'] >= start:
                result.append(d)
        return result


Exceptions
~~~~~~~~~~

Using the ``isnumeric()`` function allowed us to check for invalid user input in the specific
case above, but Python provides a far more general and powerful error handling capability, called
Exceptions. Refer back to the unit on `Error Handling <../unit03/errorhandling.html>`_ for reference.

Here's how we could deal with an invalid ``start`` parameter provided by the user
using exceptions:

.. code-block:: python3

    try:
        start = int(start)
    except ValueError:
        # return some kind of error message...

    # at this point in the code, we know the int(start) "worked" and so we are safe
    # to use it as an integer..


Here is the full code for our route function with exception handling.

.. code-block:: python3
   :linenos:

   @app.route('/degrees', methods=['GET'])
   def degrees():
       start = request.args.get('start', 0)
       try:
           start = int(start)
       except ValueError:
           return "Invalid start parameter; start must be an integer."
       data = get_data()
       result = []
       for d in data:
           if d['year'] >= start:
               result.append(d)
       return result

.. warning::

   When curling a URL with multiple query parameters separated by an ``&`` symbol,
   make sure to put the URL in quotes, e.g.:

   ``curl 'http://127.0.0.1:5000/degrees?start=1990&limit=2'``


EXERCISE 8
~~~~~~~~~~


Add support for a ``limit`` parameter to the code you wrote for Exercise 7. The
``limit`` parameter should be optional. When passed with an integer value, the
API should return no more than ``limit`` data points.


CRUD Operations
---------------

To this point, we have looked at only the ``GET`` method. There are three other
methods that are important to learn when working with REST APIs - ``PUT``, ``POST``,
and ``DELETE``. Collectively, these four methods perform **CRUD** operations on our 
data:

* **C**\ reate: ``POST`` - add a new item to a collection
* **R**\ ead: ``GET`` - get an item from a collection
* **U**\ pdate: ``PUT`` - edit an existing item in a collection
* **D**\ elete: ``DELETE`` - delete an item from a collection

To implement one of these methods into a Flask route, the method first must be 
listed in the decorator. Then, the function below the decorator must contain some
logic to act according to the method and request. The Flask ``request`` library
again comes in handy here. Suppose we want to add a ``DELETE`` method to our 
``/degrees`` route. Consider the following reduced code:


.. code-block:: python3
   :linenos:
   
   from flask import Flask, request
   
   app = Flask(__name__)
   data = [ {'id': 0, 'year': 1990, 'degrees': 5818},
            {'id': 1, 'year': 1991, 'degrees': 5725},
            {'id': 2, 'year': 1992, 'degrees': 6005},
            {'id': 3, 'year': 1993, 'degrees': 6123},
            {'id': 4, 'year': 1994, 'degrees': 6096} ]
   
   @app.route('/degrees', methods=['GET', 'DELETE'])
   def degrees():
       global data
       if request.method == 'GET':
           return(data)
       elif request.method == 'DELETE':
           content = request.get_json()
           new_data = [] 
           for item in data:
               if item['id'] != content['id']:
                   new_data.append(item)
           data = new_data
           return(data)
   
   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')


Note that data is now declared at the beginning of the script rather than returned 
via a function call. And, on line 12 we use the ``global`` keyword to indicate that 
references to ``data`` within the ``degrees()`` function (including changes) should 
be made to the variable which belongs to the global scope.

Curling this route with a ``GET`` request returns the entire list of degrees:

.. code-block:: console

   [user-vm]$ curl http://127.0.0.1:5000/degrees
   -or-
   [user-vm]$ curl -X GET http://127.0.0.1:5000/degrees

But now we are able to make a ``DELETE`` request to this route, where it is expecting
to receive a JSON data packet containing the id of the item to delete:

.. code-block:: console

   [user-vm]$ curl -X DELETE http://127.0.0.1:5000/degrees \
                   -H 'Content-Type: application/json' \
                   -d '{"id": 4}'

The important changes here are that we are now specifying a delete request 
(``-X DELETE``) instead of the default get request, the header (``-H``) flag is
used to indicate that we are sending some data in JSON format (``'Content-Type: application/json'``),
and finally the data packet that we send is just plain JSON (``-d '{"id": 4}'``).

After invoking this delete request, perform another GET request on the ``/degrees``
URL to see what has changed.
   



EXERCISE 9
~~~~~~~~~~

Add support for the ``PUT`` and ``POST`` methods to your ``/degrees`` route. 
What information should be sent in the JSON data packet? What exception handling
should be performed in the route?



Additional Resources
--------------------

* `Flask JSON support <https://flask.palletsprojects.com/en/3.0.x/api/?highlight=jsonify#module-flask.json>`_
* `Flask query parameter support <https://flask.palletsprojects.com/en/3.0.x/api/?highlight=jsonify#flask.Request.args>`_
