Messaging Systems
=================

The Queue is a powerful data structure which forms the foundation of many concurrent design patterns. Often these
design patterns center around passing messages between agents within the concurrent system. We will explore one of the
simplest and most useful of these message-based patterns - the so-called "Task Queue". By the end of this module,
students should be able to:

  * Describe the components of a task queue system, and explain how it will be utilized within our 
    Flask-based API system architecture.
  * Create task queues in Redis using the ``hotqueue`` library, and work with the ``put()`` and 
    ``consume()`` methods to queue and receive messages across two Python programs. 
  * Use the ``@q.worker`` decorator in ``hotqueue`` to create a simple Python consumer program.
  * Explain the general approach to organizing Python code into different modules and describe how to
    do this for the flask-based API system we are building. 
  * Implement good code organization practices including denoting objects as public or private. 


Task Queues
-----------

In a task queue system,

  * Agents called "producers" write messages to a queue that describe work to be done.
  * A separate set of agents called "consumers" receive the messages and do the work. While work is being done,
    no new messages are received by the consumer.
  * Each message is delivered exactly once to a single consumer to ensure no work is "duplicated".
  * Multiple consumers can be processing "work" messages at once, and similarly, 0 consumers can be processing messages
    at a given time (in which case, messages will simply queue up).

The Task Queue pattern is a good fit for our jobs service.

  * Our Flask API will play the role of producer.
  * One or more "worker" programs will play the role of consumer.
  * Workers will receive messages about new jobs to execute and performing the analysis steps.



Task Queues in Redis
--------------------

The ``HotQueue`` class provides two methods for creating a task queue consumer; the first is the ``.consume()`` method
and the second is the ``@q.worker`` decorator.


The Consume Method
~~~~~~~~~~~~~~~~~~

With a ``q`` object defined like ``q = HotQueue('queue', host='<Redis_IP>', port=6379, db=1)``,
the consume method works as follows:

  * The ``q.consume()`` method returns an iterator which can be looped over using a ``for`` loop (much like a list).
  * Each object returned by the iterator is a message received from the task queue.
  * The ``q.consume()`` method blocks (i.e., waits indefinitely) when there are no additional messages in the queue
    arbitrarily named ``queue``.
  

The basic syntax of the consume method is this:

.. code-block:: python3

   for item in q.consume():
       # do something with item

In this case, the ``item`` object is the message that was retrieved from the task queue. 


EXERCISE 1
~~~~~~~~~~

Complete the following on your JetStream VM.

  1. Start a Redis container in the background and expose port 6379.
  2. Open two separate terminals, each logged in to the user VM and running an interactive Python shell.
  3. In each terminal, create a ``HotQueue`` object pointing to the same Redis queue.
  4. In the first terminal, add three or four Python strings to the queue; check the length of the queue.
  5. In the second terminal, use a ``for`` loop and the ``.consume()`` method to print objects in the queue to the screen.
  6. Observe that the strings are printed out in the second terminal.
  7. Back in the first terminal, check the length of the queue; add some more objects to the queue.
  8. Confirm the newly added objects are "instantaneously" printed to the screen back in the second terminal.


EXERCISE 2
~~~~~~~~~~

Repeat the above steps, but this time orchestrate three containers together using ``docker-compose``: a Redis
container and two other Python containers which may simulate, for example, a Flask app and a worker. 

.. code-block:: yaml

   ---
   version: "3"

   services:
       redis-db:
           image: redis:7
           volumes:
               - ./data:/data
           ports:
               - 6379:6379
           user: "1000:1000"
           command: ["--save", "1", "1"]
       python1:
           image: python:3.10
           command: ["sleep", "9999999"]
       python2:
           image: python:3.10
           command: ["sleep", "9999999"]


Use the above ``docker-compose.yml`` file (make sure you understand what each part is doing), and execute 
the command:

.. code-block:: console

   [user-vm]$ docker-compose up -d
   Creating network "messaging_default" with the default driver
   Creating messaging_redis-db_1 ... done
   Creating messaging_python2_1  ... done
   Creating messaging_python1_1  ... done


Once the containers are running, use ``docker ps -a`` to find the names of the container, and ``docker exec``
to create two new shells inside the running containers:

.. code-block:: console

   # From terminal 1
   [user-vm]$ docker exec -it messaging_python1_1 /bin/bash
   root@ba734c20dfe3:/#

.. code-block:: console

   # From terminal 2
   [user-vm]$ docker exec -it messaging_python2_1 /bin/bash
   root@22ca40c5cf18:/# 


.. note::

   Once inside the running containers, what IP / alias do you use to refer to the Redis container?
   What libraries might you have to pip install?

When finished with the exercise, clean up your running containers by doing:

.. code-block:: console

   [user-vm]$ docker-compose down
   Stopping messaging_python2_1  ... done
   Stopping messaging_python1_1  ... done
   Stopping messaging_redis-db_1 ... done
   Removing messaging_python2_1  ... done
   Removing messaging_python1_1  ... done
   Removing messaging_redis-db_1 ... done
   Removing network messaging_default


The @q.worker Decorator
~~~~~~~~~~~~~~~~~~~~~~~

Given a HotQueue queue object, ``q``, the ``@q.worker`` decorator is a convenience utility to turn a function into a consumer
without having to write a for loop. The basic syntax is:

.. code-block:: python3

   >>> @q.worker
   >>> def do_work(item):
   >>>     # do something with item

In the example above, ``item`` will be populated with the item dequeued.

Then, to start consuming messages, simply call the function:

.. code-block:: python

    >>> do_work()
    # ... blocks until new messages arrive

.. note::

  The ``@q.worker`` decorator replaces the ``for`` loop. Once you call a function decorated with ``@q.worker``, the
  code never returns unless there is an unhandled exception.


EXERCISE 3
~~~~~~~~~~

Write a function, ``echo(item)``, to print an item to the screen, and use the ``@q.worker`` decorator to
turn it into a consumer. Call your echo function in one terminal and in a separate terminal, send messages to the
Redis queue. Verify that the message items are printed to the screen in the first terminal.

In practice, we will use the ``@q.worker`` in a Python source file like so --

.. code-block:: python

   # A simple example of Python source file, worker.py
   q = HotQueue('queue', host='<Redis_IP>', port=6379, db=1)

   @q.worker
   def do_work(item):
       # do something with item...

   do_work()


Assuming the file above was saved as ``worker.py``, calling ``python worker.py`` from the shell would result in a
non-terminating program that "processed" the items in the ``"queue"`` queue using the ``do_work(item)`` function.
The only thing that would cause our worker to stop is an unhandled exception.


Additional Resources
--------------------

* `HotQueue <https://pypi.org/project/hotqueue/>`_



