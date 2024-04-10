Homework 08
===========

**Due Date: Tuesday, Apr 16, by 11:00am central time**

Isle of Web Apps
----------------

Scenario: Continuing from the previous homework, we will add some more features
to our growing web apps. Please start from the files developed in Homework 07, and
continue to use the `HGNC data <https://www.genenames.org/download/archive/>`_.

At this point we assume you can orchestrate your Redis, Flask, and Worker services
together using docker-compose in the **development environment** (Jetstream cluster,
``[user-vm]$``). We also assume there is a rudimentary **jobs** functionality that 
takes POST requests and uses the Jobs API to initiate the jobs cycle. (Essentially
all parts of the previous homework should be working).

.. note::

   If you want to do this homework with a different data set, e.g. the data set you
   want to work on for your final project, just get it approved with the instructors
   first.




PART 1
~~~~~~

In the previous homework, we used a placeholder in the Worker script (e.g. a 15-second
sleep command) instead of actually performing any interesting work. In this homework,
update the Worker script to do some actual analysis guided by user input and pulling
from the raw data. **For Example**: Imagine the data set for this web app is the ISS
data from our midterm. A user might POST a data packet containing a 'start_epoch' and
'end_epoch' to the ``/jobs`` route. Then, the worker could pull all the raw data between
those epochs and create a plot of, e.g., average speed over time. The challenge for this
homework is to come up with a reasonable analysis for the HGNC data (or your final project
data if pre-approved), and then define what needs to be POSTed by the user to initiate that
analysis.

Create a new route in your Flask app ``/results/<jobid>``. This should be in addition to
the existing routes from previous homeworks. The new route should satisfy the following
requirements:

* A GET request to ``/results/<jobid>`` should return some information (e.g. a computed
  outcome, a plot, etc.) from a given Job ID. If the job has not yet been finished, the
  result should return a message stating that.

Results should be stored in a separate database (e.g. db=3 from our in-class diagram) and
accessed through a new client defined in the Jobs API module. Please use defensive 
programming strategies. E.g. if a user supplies an invalid Job ID, this should hopefully
not crash any of your containers.



PART 2
~~~~~~

Following our Python best practices, we will start to build out a few more elements
of our web app. Specifically, we will be looking for:

* Appropriate logging statements throughout the Flask, Worker, and Jobs API codes.
* Provide support for an environment variable to set log level. The environment 
  variable should be defined in the docker-compose file (e.g. LOG_LEVEL=WARNING) and 
  read in to the appropriate Python scripts, similar to how we read in the Redis IP
  address variable.
* Doc strings and type hints throughout the Flask, Worker, and Jobs API code. 
* Integration tests for functions that are Flask routes (see 
  `this link <../unit06/flask_special_topics.html>`_) and
  unit tests for all other appropriate functions in the Flask, Worker, and Jobs API
  code.



PART 3
~~~~~~

Write a README with the standard sections from previous homeworks: there should
be a descriptive title, there should be a high level description of the project,
there should be concise descriptions of the main files within, and you should
be using Markdown styles and formatting to your advantage. We will specifically
be looking for:

* Instructions to POST a job with data packet to ``/jobs``
* Instructions to GET and interpret results from the ``/results/<jobid>`` route
* Instructions to launch the containerized web app
* Give example API query commands and expected outputs in code blocks

As always, your README should also have a section to describe the data itself. Please
give enough information for others to understand what data they are seeing and
what it means (not every field must be described, just a general overview).
Please cite the data appropriately as well.

Finally, create a software diagram illustrating some interesting or important
part(s) of your web apps. Software diagrams should be rendered in your README
using markdown styles and sufficiently described.




What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 08 (notice the evolving organization):

.. code-block:: text
   :emphasize-lines: 7-22

   my-coe332-hws/
   ├── homework01/
   │   └── ...
   ├── ...
   ├── homework07/
   │   └── ...
   ├── homework08
   │   ├── Dockerfile
   │   ├── README.md
   │   ├── data
   │   │   └── .gitcanary
   │   ├── diagram.png
   │   ├── docker-compose.yml
   │   ├── requirements.txt
   │   ├── src
   │   │   ├── api.py
   │   │   ├── jobs.py
   │   │   └── worker.py
   │   └── test
   │       ├── test_api.py
   │       ├── test_jobs.py
   │       └── test_worker.py
   └── README.md



Additional Resources
--------------------

* `Environment Variables in Docker-compose <https://docs.docker.com/compose/environment-variables/set-environment-variables/>`_ 
* `Integration Tests in Flask <./unit06/flask_special_topics.html>`_
* Please find us in the class Slack channel if you have any questions!

