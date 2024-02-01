Error Handling
==============

Error handling is an important part of data science. Even syntactically-sound
code can run into errors if the input data is messy. Being able to understand
error messages, anticipate where errors might happen, and program defensively 
against errors will make your code more robust. After working through this 
module, students should be able to:


* Anticipate where and what types of errors might occur in their code
* Read traceback statements and identify exceptions
* Write code blocks to catch and handle exceptions



Why Do We Need Error Handling?
------------------------------

#. Prevents program from crashing if an error occurs
    * If an error occurs in a program, we don't want the program to unexpectedly
      crash on the user. Instead, error handling can be used to notify the user of
      why the error occurred and gracefully exit the process that caused the error.

#. Saves time debugging errors
    * Following reason #1, having the program display an error instead of immediately
      crashing will save a lot of time when debugging errors.
    * The logic inside the error handler can be updated to display useful information
      for the developer, such as the code traceback, type of error, etc.

#. Helps define requirements for the program
    * If the program crashes due to bad input, the error handler could notify the user
      of why the error occurred and define the requirements and constraints of the program.


`Source <https://betterprogramming.pub/handling-errors-in-python-9f1b32952423>`_


Errors in Data
--------------

Imagine you have a data set of meteorite landings in a list-of-dictionaries 
format, and that one of the descriptors for each meteorite was ``mass``. You
might reasonable expect each ``mass`` to have a value that is a positive number.

.. code-block:: python3 

   >>> from ml_data_analysis import compute_average_mass
   >>> 
   >>> data = [ {'mass': 10},
   ...          {'mass': 20},
   ...          {'mass': 40},
   ...          {'mass': 30} ]
   >>> 
   >>> compute_average_mass(data, 'mass')
   25.0

We have been through the code in ``compute_average_mass`` many times at this
point and we are pretty sure it is free of errors. But, what if the error comes 
from the data?

.. code-block:: python3 

   >>> from ml_data_analysis import compute_average_mass
   >>> 
   >>> data = [ {'mass': 10},
   ...          {'mass': 20},
   ...          {'mass': 40},
   ...          {'mass': None} ]
   >>> 
   >>> compute_average_mass(data, 'mass')
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
     File "/home/wallen/coe-332/working-with-json/ml_data_analysis.py", line 6, in compute_average_mass
       total_mass += float(item[a_key_string])
   TypeError: float() argument must be a string or a number, not 'NoneType'


At this point we need to make a choice. If we have input data which could be
10s or 100s of thousands of lines (or more), do we want to go through it and pull
out all the data points with null values for masses?

It would be better to update our code to anticipate this possible error and
handle it in a way such that our code does not crash and we still get a
result that makes sense.


Understanding Exceptions
------------------------

When errors do occur, Python3 prints a **traceback** message to help you pinpoint
where the specific **exception** occurred. With traceback messages, you generally
want to read the bottom line first, which identifies the specific exception, and
then start reading up to find out where in the code (i.e. in what function) the
exception occurred. For most errors, you can probably get away with only looking
at the last three or so lines of the traceback message.

At first glance, exceptions and traceback messages may seem to be undecipherable, 
but understanding that there are a finite number of built in exceptions and each
named exception actually is a pretty useful hint to where the error occurred. 
Consider some of the following exceptions:

.. code-block:: python3

   >>> 10 * (1/0)
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   ZeroDivisionError: division by zero
   >>> 4 + spam*3
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   NameError: name 'spam' is not defined
   >>> '2' + 2
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   TypeError: can only concatenate str (not "int") to str

ZeroDivisionError, NameError, and TypeError are somewhat self explanatory when
you see when they are raised. Knowing what circumstances can cause a built-in
exception to occur (e.g. NameErrors are raised when a name is not found) is the
first step toward identifying the cause and the solution

A list of all built-in exceptions that could occur can be found `here <https://docs.python.org/3/library/exceptions.html>`_.

.. note:: 

    Note that syntax errors stand apart as exceptions that can't be handled:

.. code-block:: python3

   >>> print 'Hello, world!'
   File "<stdin>", line 1
     print 'Hello, world!'
           ^
   SyntaxError: Missing parentheses in call to 'print'. Did you mean print('Hello, world!')?



Handling Exceptions
-------------------

Consider our meteorite landings data again. We can use a strategy called 
*exception handling* to prevent our program from crashing if it encounters bad
input data. The specific statements we use for this in Python3 are ``try`` and
``except``.

Take the original ``compute_average_mass`` function:

.. code-block:: python3
   :linenos:

   def compute_average_mass(a_list_of_dicts, a_key_string):
       total_mass = 0.
       for item in a_list_of_dicts:
           total_mass += float(item[a_key_string])
       return(total_mass / len(a_list_of_dicts) )



And update it as follows:


.. code-block:: python3
   :linenos:

   def compute_average_mass_new(a_list_of_dicts, a_key_string):
       total_mass = 0.
       num_of_valid_masses = 0
       for item in a_list_of_dicts:
           try: 
               total_mass += float(item[a_key_string])
               num_of_valid_masses += 1
           except TypeError:
               logging.warning(f'encountered non-float value {item[a_key_string]} in compute_average_mass')
       return(total_mass / num_of_valid_masses)


What happens here is that the lines inside the ``try`` block are executed. If no
exception is raised, then the ``except`` block is skipped and the code continues to
the next iteration of the for loop.

If a ``TypeError`` is raised in the ``try`` block, (i.e. beause ``item[a_key_string]``
is not a float) then that exception is handled by
executing the lines in the ``except`` block. In this case, a message is logged and
the code continues to the next iteration of the for loop. If any other kind of error
occurs, the program would raise the error and exit with a traceback message. Here,
we are currently only handling ``TypeErrors``. 

With this modified function, we can execute our lines of code from above:

.. code-block:: python3 

   >>> from ml_data_analysis import compute_average_mass_new
   >>> 
   >>> data = [ {'mass': 10},
   ...          {'mass': 20},
   ...          {'mass': 40},
   ...          {'mass': None} ]
   >>> 
   >>> compute_average_mass_new(data, 'mass')
   WARNING: encountered non-float value None in compute_average_mass_new
   23.333333333333332

See the resources below for tips on building more complicated try-except statements
that can handle multiple different exceptions, and use the ``finally`` statement
to execute code after the block whether an exception was raised or not.

Additional Resources
--------------------

* `Python 3 Error Handling <https://docs.python.org/3/tutorial/errors.html>`_
* `Python 3 Exception Class <https://docs.python.org/3/library/exceptions.html>`_



