Homework 04
===========

**Due Date: Tuesday, Feb 20, by 11:00am central time**

The ISS Aquatic
---------------

Scenario: You have found an abundance of interesting positional and velocity
data for the International Space Station (ISS). It is a challenge, however, to
sift through the data manually to find what you are looking for. Your long-term
objective is to build a web app for querying and returning interesting
information from the ISS data set. For this assignment, we will just start to
assemble necessary functions and containerize the tool.


PART 1
~~~~~~

First, navigate to the 
`ISS Trajectory Data <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
website. 

There are two downloads available, and they contain the same data. It is the 
'Orbital Ephemeris Message (OEM)' data in either plain text or XML format.
Download and examine both files to get a feel for what they contain. Please
closely read the ~3 paragraphs of accompanying text on the website as well.

The most important thing to understand is that the data contains ISS state
vectors over a 15 day period. The range of days may change depending on
when you download the data set. 'State vectors' are the Cartesian vectors
for both position ``{X, Y, Z}`` and velocity ``{X_DOT, Y_DOT, Z_DOT}`` that,
along with a time stamp ``(EPOCH)``, describe the complete state of the system
(the ISS).  The frame of reference for these vectors is Earth, based on something
called the J2000 reference frame.

.. tip::

   The XML data is probably a little bit easier than the raw text data to
   ingest into dictionary format inside a python script. See the 
   `Unit on XML <../unit02/xml.html>`_ for tips.


PART 2
~~~~~~

Write a (containerized) Python3 script that does the following:

1) Ingests the ISS data using the ``requests`` library, and stores it in
   something similar to a list-of-dictionaries format
2) Prints a statement about the range of data using timestamps from the 
   first and last epochs
3) Prints the full epoch (time stamp, state vectors, and velocities)
   closest to "now" - or the time when the program is executed. This
   will change based on when the program is run
4) Calculates and prints the average speed over the whole data set, as
   well as the instantaneous speed closest to "now".


All of the normal Python3 script best practices apply:

* Write appropriately formatted doc strings for your routes
* Use type annotations where appropriate
* Organize your code in the same way as the other scripts we have written
* Write unit tests for all non ``main()`` functions
* Implement selective logging and error handling where appropriate

.. tip::

   For the speed function, you will need an equation to calculate speed from
   Cartesian velocity vectors: **speed = sqrt(x_dot^2 + y_dot^2 + z_dot^2)**



PART 3
~~~~~~

The homework must also include a README file. The README should be descriptive,
use proper grammar, and contain enough instructions so anyone else could clone
the repository and figure out how to run your app and interact with it. 
Guidelines to follow for the README are:

* Descriptive title
* High-level description of the folder contents / project objective. I.e. why
  does this exist and why is it important? (2-3 sentences)
* Instructions on how to access and description of the data set from the original source
  (do not include the data itself in this repository!)
* Instructions on how to build a container for your code
* Instructions to run the containerized Python3 script and unit tests
* List of what to expect for output and how to interpret them
* Try to use markdown styles to your advantage, give the sections headers, use
  code blocks where appropriate, etc.

Remember, the README is your chance to document for yourself and explain to others
why the project is important, what the code is, and how to use it / interpret
the outputs / etc. This is a *software engineering and design* class, so we are
not just checking to see if your code works. We are also evaluating the design of
the overall submission, including how well the project is described in the README.




What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework03:

.. code-block:: text
   :emphasize-lines: 8-12

   my-coe332-hws/
   ├── homework01
   │   └── ...
   ├── homework02
   │   └── ...
   ├── homework03
   │   └── ...
   ├── homework04
   │   ├── Dockerfile
   │   ├── README.md
   │   ├── iss_tracker.py
   │   └── test_iss_tracker.py
   └── README.md





Additional Resources
--------------------

* `NASA Data Set <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
* `Info on State Vectors <https://en.wikipedia.org/wiki/Orbital_state_vectors>`_
* `Info on Reference Frame <https://en.wikipedia.org/wiki/Earth-centered_inertial>`_
* `Unit on XML <../unit02/xml.html>`_
* Please find us in the class Slack channel if you have any questions!

