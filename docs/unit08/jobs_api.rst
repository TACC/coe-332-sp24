The Jobs API
============

We have been introduced to the concept of *concurrency*: a method for managing resources such that multiple agents or
components of the system can be in progress at the same time without impacting the correctness of the system.
We have also discussed the utility of *asynchronicity*: an approach in concurrency wherein we can schedule a task,
receive an immediate response, and continue on to other tasks while the previous task works in the background. The
tools we will use to achieve this in the software systems we are building include worker containers, a messaging
system, a task queue, and now the *Jobs API*. 


The basic idea is that we will have a new endpoint in our API at a path ``/jobs`` (or something similar). A user wanting
to have our system perform a long-running task will create a new job by making an HTTP POST request to ``/jobs``,
describing the job in the POST message body (in JSON). Instead of performing the actual computation, the request will
simply be recorded in Redis and a response will be immediately provided to the user. The response will not include the
result of the job itself, but instead it will indicate that the request has been received and it will be worked on once
it gets to the top of the queue. Also, critically, the  response will include an ``id`` for the job so that the user can
check the status and, eventually, get the actual result. *The Jobs API is a Python module that we will write which includes
methods and tools for managing jobs in our software system.*

By the end of this module, students should be able to:

  * Explain the purpose and reasoning behind all variables and methods in Jobs API
  * Decide which variables and methods should be private and which should be public
  * Organize code for software system into API, worker, and jobs modules
  * Import the Jobs API into other modules to use for jobs functionality
  * Perform appropriate ``curl`` requests to POST jobs and GET the result of jobs



Concurrency in the Jobs API
---------------------------
Recall that our big-picture goal is to add a Jobs endpoint to our Flask system that can process long-running tasks.
We will implement our Jobs API with concurrency in mind. The goals will be:

The overall architecture will thus be:

  1. Save the request in a database and respond to the user that the analysis will eventually be run.
  2. Give the user a unique identifier with which they can check the status of their job and fetch the results when
     they are ready,
  3. Queue the job to run so that a worker can pick it up and run it.
  4. Build the worker to actually work the job.

Parts **1-3**  are the tasks of the Flask API, while part **4** will be a worker, running as a separate container,
that is waiting for new items in the Redis queue.



Code Organization
-----------------

As software systems get larger, it is very important to keep code organized so that finding the functions, classes,
etc. responsible for different behaviors is as easy as possible. To some extent, this is technology-specific, as
different languages, frameworks, etc., have different rules and conventions about code organization. We'll focus on
Python, since that is what we are using.

The basic unit of code organization in Python is called a "module". This is just a Python source file (ends in a ``.py``
extension) with variables, functions, classes, etc., defined in it. We've already used a number of modules, including
modules that are part of the Python standard library (e.g. ``json``) and modules that are part of third-party libraries
(e.g., ``redis``).

The following should be kept in mind when designing the modules of a larger system:

  * Modules should be focused, with specific tasks or functionality in mind, and their names (preferably, short)
    should match their focus.
  * Modules are also the most typical entry-point for the Python interpreter itself, (e.g., ``python some_module.py``).
  * Accessing code from external modules is accomplished through the ``import`` statement.
  * Circular imports will cause errors - if module A imports an object from module B, module B cannot import from module A.



Module Design
-------------

The Python standard library is a good source of examples of module design. You can browse the
standard library for Python 3.10 `here <https://docs.python.org/3.10/library/>`_.

  * We see the Python standard library has modules focused on a variety of computing tasks; for example, for working
    with different data types, such as the ``datetime`` module and the ``array`` module.  The descriptions are succinct:

    * *The datetime module supplies classes for manipulating dates and times.*
    * *This module defines an object type which can compactly represent an array of basic values: characters, integers, floating point numbers*

  * For working with various file formats: e.g., ``csv``, ``configparser``
  * For working with concurrency: ``threading``, ``multiprocessing``, etc.


With this in mind, a first approach might be to break up our system into two modules:

  * ``api.py`` - this module contains the Flask web server.
  * ``worker.py`` - this module contains the code to execute jobs.

However, both the API server and the workers will need to interact with the database and the queue:

  * The API will create new jobs in the database, put new jobs onto the queue, and retrieve the status of jobs
    (and probably the output products of the job).
  * The worker will pull jobs off the queue, retrieve jobs from the database, and update them.

This suggests a different structure:

  * ``api.py`` - this module contains the Flask web server.
  * ``worker.py`` - this module contains the code to execute jobs.
  * ``jobs.py`` - this module contains core functionality for working with jobs in Redis (and on the queue).


Common code for working with ``redis``/``hotqueue`` can go in the ``jobs.py`` module and be imported in both ``api.py``
and ``worker.py``.

.. note::

  High-quality modular design is a crucial aspect of building good software. It requires significant thought and
  experience to do correctly, and when done poorly it can have dire consequences. In the best case, poor module
  design can make the software difficult to maintain/upgrade; in the worst case, it can prevent it from running
  correctly at all.

We can sketch out our module design by making a list of the functionality that will be available 
in each module. This is only an initial pass at listing the functionality needed -- we will refine it 
over time -- but making an initial list is important for thinking through the problem. 

``api.py``: This file will contain all the functionality related to the Flask web server, and will 
include functions related to each of the API endpoints in our application. 

  * POST /data -- Load the data into the application. Will write to Redis.
  * GET /data?search=... -- List all of the data in the system, optionally filtering with a search
    query parameter. Will read from Redis.
  * GET /data/<id> -- Get a specific object from the dataset using its ``id``. Will read from Redis.

  * POST /jobs -- Create a new job. This function will save the job description to Redis and add a 
    new task on the queue for the job. Will write to Redis and the queue. 
  * GET /jobs -- List all the jobs. Will read from Redis. 
  * GET /jobs/<id> -- Get the status of a specific job by id. Will read from Redis. 
  * GET /jobs/<id>/results -- Return the outputs (results) of a completed job. Will read from Redis. 

``worker.py``: This file will contain all of the functionality needed to get jobs from the task
queue and execute the jobs. 

  * Get a new job -- Hotqueue consumer to get an item off the queue. Will get from the queue and 
    write to Redis to update the status of the job.
  * Perform analysis -- 
  * Finalize job -- Saves the results of the analysis and updates the job status to complete. Will
    write to Redis. 

``jobs.py``: This file will contain all functionality needed for working with jobs in the Redis 
database and the Hotqueue queue. 

  * Save a new job -- Will need to write to Redis.
  * Retrieve an existing job - Will need to read from Redis. 
  * Update an existing jobs -- Will need to read and write to Redis.  


Private vs Public Objects
-------------------------
As software projects grow, the notion of public and private access points (functions, variables, etc.) becomes an increasingly
important part of code organization.

  * Private objects should only be used within the module they are defined. If a developer needs to change the
    implementation of a private object, she only needs to make sure the changes work within the existing module.
  * Public objects can be used by external modules. Changes to public objects need more careful analysis to understand
    the impact across the system.

Like the layout of code itself, this topic is technology-specific. In this class, we
will take a simplified approach based on our use of Python. Remember, this is a simplification to illustrate the basic
concepts - in practice, more advanced/robust approaches are used.

  * We will name private objects starting with a single underscore (``_``) character.
  * If an object does not start with an underscore, it should be considered public.

We can see public and private objects in use within the standard library as well. If we open up the source code for the
``datetime`` module, which can be found `on GitHub <https://github.com/python/cpython/blob/3.9/Lib/datetime.py>`_ we see a mix
of public and private objects and methods.

  * Private objects are listed first.
  * Public objects start on `line 442 <https://github.com/python/cpython/blob/3.10/Lib/datetime.py#L473>`_ with
    the ``timedelta`` class.



EXERCISE 1
~~~~~~~~~~

Create three files, ``api.py``, ``worker.py``, and ``jobs.py`` in your local directory. You may wish to start from the
files you prepared for Homework 06. You should also have a ``Dockerfile``, ``docker-compose.yml``, and ``requirements.txt``
in this directory to help with containerization and orchestration.

.. code-block:: console
  
  [user-vm] $ ls 
  Dockerfile  api.py  docker-compose.yaml  jobs.py  requirements.txt  worker.py


Add the following function and variable definitions to ``jobs.py``. Closely examine each line to make sure you understand
the purpose. Carefully consider which are public and private, and why.


.. code-block:: python
   :linenos:

   import json
   import uuid
   import redis
   from hotqueue import HotQueue
   
   _redis_ip='redis-db'
   _redis_port='6379'
   
   rd = redis.Redis(host=_redis_ip, port=6379, db=0)
   q = HotQueue("queue", host=_redis_ip, port=6379, db=1)
   jdb = redis.Redis(host=_redis_ip, port=6379, db=2)
   
   def _generate_jid():
       """
       Generate a pseudo-random identifier for a job.
       """
       return str(uuid.uuid4())
   
   def _instantiate_job(jid, status, start, end):
       """
       Create the job object description as a python dictionary. Requires the job id,
       status, start and end parameters.
       """
       return {'id': jid,
               'status': status,
               'start': start,
               'end': end }
   
   def _save_job(jid, job_dict):
       """Save a job object in the Redis database."""
       jdb.set(jid, json.dumps(job_dict))
       return
   
   def _queue_job(jid):
       """Add a job to the redis queue."""
       q.put(jid)
       return
   
   def add_job(start, end, status="submitted"):
       """Add a job to the redis queue."""
       jid = _generate_jid()
       job_dict = _instantiate_job(jid, status, start, end)
       _save_job(jid, job_dict)
       _queue_job(jid)
       return job_dict
   
   def get_job_by_id(jid):
       """Return job dictionary given jid"""
       return json.loads(jdb.get(jid))
   
   def update_job_status(jid, status):
       """Update the status of job with job id `jid` to status `status`."""
       job_dict = get_job_by_id(jid)
       if job_dict:
           job_dict['status'] = status
           _save_job(jid, job_dict)
       else:
           raise Exception()


EXERCISE 2
~~~~~~~~~~

Write a skeleton for a Flask app in the file ``api.py``. The Flask app should:

  1. Import necessary modules, including some from ``jobs.py``
  2. Declare an instance of the Flask class
  3. Support a route for POSTing a new job
  4. Support a route for GETting job status

.. tip::

   A job POST request might look like:

   .. code-block:: console

      curl localhost:5000/jobs -X POST -d '{"start":1, "end":2}' -H "Content-Type: application/json"
    
   In this example, we are sending a 'start' and 'end' index which is important for the "work". E.g. perhaps
   the worker is designed to plot all the values between 'start' and 'end'. In practice, the app that you 
   develop may require different parameters.


EXERCISE 3
~~~~~~~~~~

Write a skeleton for a worker in the file ``worker.py``: The worker should:

  1. Import necessary modules, including some from ``jobs.py``
  2. Pull items (job IDs) off the queue
  3. When it starts working on a new job, update the job status to 'in progress'
  4. Do work (e.g. sleep for 15 seconds)
  5. When it finishes working on a new job, update the job status to 'complete'


EXERCISE 4
~~~~~~~~~~

Fill out the contents of the ``Dockerfile``, ``docker-compose.yml``, and ``requirements.txt`` in order to help with
containerization and orchestration. Pay careful attention to how you set up and build the containers. Should we be
using one Docker image or two? What should the entrypoint be? 


EXERCISE 5
~~~~~~~~~~

Modify the definition of the ``rd``, ``q``, and ``jdb`` objects to not use a hard-coded IP address,
but to instead read the IP address from an environment variable, ``REDIS_IP``. Determine how to set the value of
``REDIS_IP`` in the ``Dockerfile`` and / or ``docker-compose.yml`` file.




