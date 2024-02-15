Homework 03
===========

**Due Date: Tuesday, Feb 13, by 11:00am central time**

The Royal Containers
--------------------

The goal this week is to take your code from last week (all Python3 scripts
including unit test scripts), containerize them, and come up with a reasonable
software diagram to describe the whole project.


PART 1
~~~~~~

First copy your Python3 scripts (including unit tests) from homework02 into a
new homework03 folder. If you find that there are any errors or problems with any of your
scripts from homework02, now would be a good time to fix them.

Next, write a Dockerfile to containerize all of the scripts and unit tests that
you copied over. The Dockerfile should not pull the Meteorite Landing data into
the container - instead you should plan to volume mount the data inside the
container at run time. The Dockerfile needs to install all dependencies as well
(e.g. pytest).


PART 2
~~~~~~

Include a software diagram in your repository. The diagram should capture as
many components of your system / project as possible (e.g. the user, the VM,
the container, the scripts, the functions, the unit tests, the input data, the
output results, etc.). There is really no wrong answer here - we just want to
take a first pass at a rough diagram that we can iterate on later.


PART 3
~~~~~~

The README file can be copied from homework02, but will need modifications. The
README should describe how a user who has just cloned your repository can run your
tool in a container from start to finish. Take special care to describe how to:

* Build the image from a Dockerfile
* Get the input data from the web (data should be cited)
* Mount the data inside the container at run time
* Run the containerized code
* Run the containerized unit tests

Please also use the appropriate markdown tags in your README to display your 
image as part of your README file. Include a brief 2-3 sentence description
of what your image illustrates.


What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework03:

.. code-block:: text
   :emphasize-lines: 6-13

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── homework02
   │   └── ...
   ├── homework03
   │   ├── Dockerfile
   │   ├── README.md
   │   ├── diagram.png
   │   ├── gcd_algorithm.py
   │   ├── ml_data_analysis.py
   │   ├── test_gcd_algorithm.py
   │   └── test_ml_data_analysis.py
   └── README.md



Additional Resources
--------------------

* `Meteorite Landings Data <https://data.nasa.gov/Space-Science/Meteorite-Landings/gh4g-9sfh/about_data>`_
* `Markdown Syntax <https://www.markdownguide.org/basic-syntax/>`_
* `Tips on Writing a Good README <https://www.makeareadme.com/>`_
* `Software Diagram Tools <../unit04/diagrams.html>`_
* `Best Practices for Writing Dockerfiles <https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>`_
* Please find us in the class Slack channel if you have any questions!
