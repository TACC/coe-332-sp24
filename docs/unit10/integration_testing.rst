Integration Testing
===================

Unlike unit tests, integration tests exercise multiple components, functions, or
units of a software system at once. Some properties of integration tests include:

* Each test targets a higher-level capability, requirement or behavior of the
  system, and exercises multiple components of the system working together.
* Broader scope means fewer tests are required to cover the entire application/system.
* A given test failure provides more limited information as to the root cause.

It's worth pointing out that our definition of integration test leaves some
ambiguity. You will also see the term "functional tests" used for tests that
exercise entire aspects of a software system. After going through this module,
students should be able to:

* Identify Python frameworks for integration testing
* Identify aspects of a software system that should be tested with integration testing
* Use the Python requests library to interact with the API of your software system
* Write and execute useful integration tests using ``pytest`` and ``assert`` statements




Challenges When Writing Integration Tests
-----------------------------------------

Integration tests against large, distributed systems with lots of components
that interact face some challenges.

* We want to keep tests independent so that a single test can be run without its
  result depending on other tests.
* Most interesting applications change "state" in some way over time; e.g., files
  are saved/updated, database records are written, queue systems updated. In order
  to properly test the system, specific state must be established before and after
  a test (for example, inserting a record into a database before testing the
  "update" function).
* Some components have external interactions, such as an email server,
  a component that makes an update in an external system (e.g. GitHub) etc. A
  decision has to be made about whether or not this functionality will be
  validated in the test and if so, how.



Initial Integration Tests for Our Flask API
-------------------------------------------

For our first set of integration tests, we'll use the following strategy:

* Start the Flask API, Redis DB, and Worker services
* Use the ``requests`` library in our test scripts to make requests directly to
  the running API server
* Check various aspects of the response; each check can be done with a simple
  ``assert`` statement, just like for unit tests

.. note::

   In large software projects you may see unit tests and integration tests
   separated into different tests scripts. The projects in this class are 
   small enough and the tests will run quickly enough that it is generally
   fine to just keep everything together.


Organizing Test Files
~~~~~~~~~~~~~~~~~~~~~

As we have seen before, test scripts should be named strategically and organized
into a subdirectory similar to:

.. code-block:: text

   my-api/
   ├── data
   │   └── .gitcanary
   ├── docker-compose.yml
   ├── Dockerfile
   ├── kubernetes
   │   ├── ...
   │   └── ...
   ├── requirements.txt
   ├── src
   │   ├── flask_api.py
   │   ├── jobs.py
   │   └── worker.py
   └── test
       ├── test_flask_api.py
       ├── test_jobs.py
       └── test_worker.py

The simplest way to import target methods into your test scripts is to containerize
the test scripts in the same directory as your src files. For example, a few lines
of your Dockerfile could resemble:

.. code-block:: Dockerfile

   ...
   WORKDIR /app
   COPY src/* ./
   COPY test/* ./
   ...

Then, your test scripts could import directly from your src files without
needing to account for relative path For unit tests, you will need to import the target
method. For integration tests, you likely can get away without importing the target
method and instead just using the ``requests`` library. 


A Simple pytest Example
~~~~~~~~~~~~~~~~~~~~~~~

Assume you have a simple Flask app in the file ``flask_api.py`` with a 'Hello, world!'
route like:

.. code-block:: python3

   @app.route('/hello', methods=['GET'])                             
   def hello():                                         
       return 'Hello, world!\n'

You could approach this function from a unit test point of view and write a simple
check to make sure it returns the correct string. In ``test_hello_flask.py`` you
would have lines of code similar to:

.. code-block:: python3

   from flask_api import hello

   def test_hello():                      
     assert (hello()) == 'Hello, world!\n'
                                     
   
However, to test the Flask app as it is integrated into the overall software 
system, it would be useful to approach this test from a different angle:

.. code-block:: python3

   import requests

   def test_integration_hello():
       response = requests.get('http://localhost:5000/hello')
       assert response.status_code == 200

Notice this time we do not need to import the ``hello()`` method directly.
This small test just checks to make sure curling the route (with the Python
requests library) returns a successful status code, ``200``. In practice, it
may be useful to keep both tests.


Running Containerized Tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To run the tests in a container, first ensure that the test scripts are
containerized along side the src code. Before running the tests, however,
it is important to understand more about Docker networks. When you perform
``docker-compose up``, Docker creates a bridge network such that different
services that are part of the same docker-compose.yml file can communicate
with each other using service names as host aliases. This is what allows us
to use, e.g. ``redis-db`` as the IP address for the Redis container, so long
as the database service is named ``redis-db`` in the docker-compose.yml file.

To run containerized tests, we need to (1) ensure that the container running
the tests is on the same bridge network as the services and (2) inject a 
hostname alias for the Flask API into the test file. 

First perform docker-compose up and identify the name of the bridge network:

.. code-block:: console

   [user-vm]$ docker-compose up --build -d
   ...
   [user-vm]$ docker network ls
   ...
   eea9458e6e9f   demo_default        bridge    local
   ...

There are typically several networks. Look for one with the same name as the 
folder your files are in. You can also use ``docker inspect`` on a running 
container to see what network it is on. Once you identify the network, you
need to launch the test container on the same network using a flag:

.. code-block:: console

   [user-vm]$ docker run --rm --network demo_default image:tag pytest
   ============================= test session starts ==============================
   platform linux -- Python 3.10.14, pytest-8.1.1, pluggy-1.4.0
   rootdir: /app
   collected 2 items

   test_hello_flask.py ..                                                   [100%]

   ============================== 2 passed in 0.20s ===============================

Remember, the tests will become a little more flexible if you create a way to
dynamically set the Flask API hostname. (We did something similar for the Redis
IP address in a previous lesson). Imaging updating the test script(s) to include
something similar to the following:

.. code-block:: python3
   :linenos:

   import requests
   import os
   from flask_api import hello

   _flask_ip=os.environ.get('FLASK_IP')

   def test_hello():                      
     assert (hello()) == 'Hello, world!\n'

   def test_integration_hello():
       response = requests.get(f'http://{_flask_ip}:5000/hello')
       assert response.status_code == 200

QUESTIONS
~~~~~~~~~

* How do we set the value of ``FLASK_IP`` in the environment?
* What should the value be set to?


EXERCISE
~~~~~~~~

Continue working in the test file, ``test_api.py``, and write a new functional
test that use the ``requests`` library to make a ``GET`` request to the ``/jobs``
endpoint and check the response for, e.g.:

* The response returns a 200 status code
* The response returns a valid JSON string
* The response can be decoded to a Python dictionary


Remember, your services should be running and as much as possible, functional tests
should be testing the end-to-end functionality of your entire app.
