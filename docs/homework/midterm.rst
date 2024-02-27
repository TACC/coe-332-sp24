Midterm Project
===============

**Due Date 1: Friday, Mar 8, by 11:59 pm central time**

**Due Date 2: Monday, Mar 18, by 11:59 pm central time**

Projects turned in by due date 1 will receive +10% on midterm (+2% toward 
final grade). Projects will not be accepted after due date 2.


Fantastic Mr. ISS
-----------------

In Homework 05, we turned our ISS tracker into a proper Flask app. Now,
we will finish off this project by adding a few final routes, automate deployment
with Docker Compose, and write up a short document describing the project.


PART 1
~~~~~~

For the Midterm Project, start a brand new git repository. Copy over the files
that you have been working on from Homework 05. Your current iteration of
``iss_tracker.py`` should have seven functional Flask routes as described in
`Homework 05 <./homework05.html>`_. For this project, keep the existing
routes and add the following new routes:

* A ``/comment`` route that returns the 'comment' list object from the ISS data
* A ``/header`` route that returns the 'header' dictionary object from the ISS data
* A ``/metadata`` route that returns the 'metadata' dictionary object from the ISS data
* A ``/epochs/<epoch>/location`` route that returns latitude, longitude, altitude, and 
  geoposition for a given ``<epoch>``. Math required! See slack for hints on this. We
  will also be using the `Python GeoPy <https://geopy.readthedocs.io/en/stable/#>`_
  library for determining geoposition given a latitude and longitude.
* A ``/now`` route that returns the same information as the 'location' route above, but
  for the up-to-date, real time position of the ISS. You will need to iterate over all
  Epochs in the data set and figure out which one is closest to the current time, then
  return location information for that Epcoh. You can confirm whether it is working by
  comparing it to the longitude and latitude of the ISS in real time using an
  `online tracker <https://www.n2yo.com/?s=90027>`_.

After completing the above, your app should have eight total routes, one of which
supports query parameters:

+----------------------------------+------------+---------------------------------------------+
| **Route**                        | **Method** | **What it should do**                       |
+----------------------------------+------------+---------------------------------------------+
| ``/comment``                     | GET        | Return 'comment' list object from ISS data  |
+----------------------------------+------------+---------------------------------------------+
| ``/header``                      | GET        | Return 'header' dict object from ISS data   |
+----------------------------------+------------+---------------------------------------------+
| ``/metadata``                    | GET        | Return 'metadata' dict object from ISS data |
+----------------------------------+------------+---------------------------------------------+
| ``/epochs``                      | GET        | Return entire data set                      |
+----------------------------------+------------+---------------------------------------------+
| ``/epochs?limit=int&offset=int`` | GET        | Return modified list of Epochs given query  |
|                                  |            | parameters                                  |
+----------------------------------+------------+---------------------------------------------+
| ``/epochs/<epoch>``              | GET        | Return state vectors for a specific Epoch   |
|                                  |            | from the data set                           |
+----------------------------------+------------+---------------------------------------------+
| ``/epochs/<epoch>/speed``        | GET        | Return instantaneous speed for a specific   |
|                                  |            | Epoch in the data set (math required!)      |
+----------------------------------+------------+---------------------------------------------+
| ``/epochs/<epoch>/location``     | GET        | Return latitude, longitude, altitude, and   |
|                                  |            | geoposition for a specific Epoch in the     |
|                                  |            | data set (math required!)                   |
+----------------------------------+------------+---------------------------------------------+
| ``/now``                         | GET        | Return instantaneous speed, latitude,       |
|                                  |            | longitude, altitude, and geoposition for    |
|                                  |            | the Epoch that is nearest in time           |
+----------------------------------+------------+---------------------------------------------+

Please use defensive programming strategies for your routes. Assume we will try to
break your routes by sending bad information to your URL and query parameters
Your routes should use exception handling to catch and handle these bad requests.
In addition:

* Write appropriately formatted doc strings for your routes
* Use type annotations where appropriate
* Organize your code in the same way as the other Flask apps we have made
* Write appropriate unit tests to test each function



PART 2
~~~~~~

**requirements.txt:** Our Python libraries are growing in number and complexity.
Write a ``requirements.txt`` file that lists all the non-standard Python libraries that
must be installed for this project.

**Dockerfile:** Write a Dockerfile to containerize your ``iss_tracker.py`` script. 
Make sure it installs any Python libraries required from ``requirements.txt``.
The default command should launch the Flask app in a standard way.

**docker-compose.yml:** Write a compose file to automate the deployment of your app. The 
compose file should be configured to build the image, and to bind the appropriate port
inside the container to the appropriate port on the host.



PART 3
~~~~~~

**README.** Standard README guidelines apply. The README should have a descriptive title
(not “Midterm Project”), a short summary / description of the project, short descriptions
of any very important files that the reader should know about. We will specifically be looking 
for:

* A citation of the ISS data
* Instructions to deploy the app with docker compose
* Instructions to curl all routes, plus what the outputs meaning
* Instructions to run the containerized unit tests

Be succinct, use Markdown styles, and show example commands / expected output.


**Diagram.png:** Include a software diagram that illustrates what you deem to be
the most important parts of your project. We will share sample diagrams in class
to show what would be reasonable. 


**Write Up.** You must also turn in a written document (as a PDF) describing the project.
While the README is typically more succinct and targeted towards developers / users of
the application, the written document should be more verbose and targeted towards a non-user,
but technically savvy layperson. For example, in the README you might instruct a user: “Run
the application by performing xyz”. In the Write Up, you would instead describe what users do:
“Users of the application can run it by performing xyz”. In other words, write this document
as if you are describing to your fellow engineering students what you did in this class for
your Midterm project.

In the Write Up, include some narrative about the motivation of the project and why it is an
interesting or important application to have. You must also include a short section on “Ethical
and Professional Responsibilities in Engineering Situations”. As a suggestion, you may consider
describing the importance of citing data or the importance of verifying the quality and accuracy
of data that goes into applications such as these. We strongly encourage you to come up with other
Ethical and Professional Responsibilities that might apply here.

Please make sure to appropriately cite the data source in both the README and the written document.


What to Turn In
---------------

This Midterm project should be pushed into a standalone repo with a descriptive
name (not “coe332-midterm”). It should not be put into your existing homework repo.
A sample Git repository should contain the following after completing the Midterm:

.. code-block:: text

   ISS-Tracker/                
   ├── Dockerfile
   ├── README.md
   ├── diagram.png
   ├── docker-compose.yml
   ├── iss_tracker.py
   ├── requirements.txt
   └── test
       └── test_iss_tracker.py    # put unit tests in a sub folder

Send an email to wallen@tacc.utexas.edu with the written PDF summary of the project
attached plus a link to your new GitHub repository. Please include “Midterm Project”
in the subject line. We will clone all of your repos at the due date / time for evaluation.


.. note::
  
   Do not include the raw  data as part of your repo.



Additional Resources
--------------------

* `NASA Data Set <https://spotthestation.nasa.gov/trajectory_data.cfm>`_
* `Real Time ISS Position <https://www.n2yo.com/?s=90027>`_
* `API for Real Time ISS Position <http://api.open-notify.org/iss-now.json>`_
* `Python GeoPy Docs <https://geopy.readthedocs.io/en/stable/#>`_
* Please find us in the class Slack channel if you have any questions!
