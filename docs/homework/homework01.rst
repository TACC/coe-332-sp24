Homework 01
===========

**Due Date: Tuesday, Jan 30, by 11:00am central time**

Bottle Repository
-----------------

Your task is to set up a new Git repository that will be used to turn in 
your homework assignments for the semester. 

Here are the overall requirements for the repository:

* Must contain a top level ``README.md`` with a simple description of what the
  repository is for
* Must contain a directory called ``homework01``
* Must be pushed to GitHub, must be private, and must be shared with GitHub user
  wjallen / wallen@tacc.utexas.edu


Here are the requirements for the ``homework01`` directory: 

* The directory ``homework01`` must contain four Python scripts:

  * ``ml_csv_reader.py``: reads ``Meteorite_Landings.csv`` and prints summary statistics
  * ``ml_json_reader.py``: reads ``Meteorite_Landings.json`` and prints summary statistics
  * ``ml_xml_reader.py``: reads ``Meteorite_Landings.xml`` and prints summary statistics
  * ``ml_yaml_reader.py``: reads ``Meteorite_Landings.yaml`` and prints summary statistics

* The definition of "summary statistics" is open to interpretation
* You may hard-code the raw data file names into the Python scripts, but using a
  utility like argparse to accept the file name as an argument on the command line
  would be preferred
* The directory ``homework01`` must also contain a ``README.md`` file with a
  descriptive title and short descriptions of each of the four Python scripts, 
  including instructions on how to run the scripts and interpret the output

A sample Git repository should contain the following files after completing
homework 01:

.. code-block:: text

   my-coe332-hws/
   ├── homework01
   │   ├── README.md       # specifically describes the contents of the homework01 directory
   │   ├── ml_csv_reader.py
   │   ├── ml_json_reader.py
   │   ├── ml_xml_reader.py
   │   └── ml_yaml_reader.py
   └── README.md           # generally describes the whole repo

.. note::

    Do not include the raw Meteorite Landing data in your repository!
    Stick to only the files above.

What to Turn In
---------------

Send an email to wallen@tacc.utexas.edu with the link to
your GitHub repository and include "Homework 01" in the subject line. We will
clone all of your repos at the due date / time for evaluation.


Additional Resources
--------------------

* `Meteorite Landing Data <https://github.com/TACC/coe-332-sp24/tree/main/docs/unit02/sample-data>`_
* Please find us in the class Slack channel if you have any questions!
