Plotting with Matplotlib
========================

Matplotlib a graphing library for Python. It has a nice collection of tools that you can use to
create anything from simple graphs, to scatter plots, to 3D graphs. It is used heavily in 
the scientific Python community for data visualization.


Using Matplotlib
----------------

.. tip::

   An easy place to test the examples below is in a browser-based
   `Jupyter notebook <https://jupyter.org/>`_


First install matplotlib with pip, or add it to your requirments.txt file. 
Installing matplot lib will install some other dependencies as well, including
numpy which is a library with support for arrays and high-level mathematical
functions.

.. code-block:: console

   [user-vm]$ pip install matplotlib


If you generate plots on a Linux VM (e.g. Jetstream), you typically won't be able
to view them on that VM unless you enable `X-forwarding <https://kb.iu.edu/d/bdnt>`_.
It takes a bit of setting up, but the alternative is to copy images to your laptop
where you can open them directly.

Simple Plots
------------

Plot a simple sin wave:

.. code-block:: python3

   import matplotlib.pyplot as plt
   import numpy as np

   x = np.linspace(0, 2*np.pi, 50)
   plt.plot(x, np.sin(x))
   plt.savefig('my_sinwave.png')  # writes a file you can download
   plt.show()                     # displays the file if in graphics-supporting environment


Continuing from the same file, plot two graphs on the same axis:

.. code-block:: python3

   plt.plot(x, np.sin(x), x, np.sin(2*x))
   plt.savefig('my_sinwavex2.png')    
   plt.show()

Change the colors and add markers:

.. code-block:: python3

   plt.plot(x, np.sin(x), 'r-o', x, np.sin(2*x), 'g--')
   plt.savefig('my_sinwavex2a.png')
   plt.show()
    
Matplot lib supports some built-in colors, and others can be accessed using 
HTML hex strings (e.g. ``'#eeefff'``):

* Blue - 'b'
* Green - 'g'
* Red - 'r'
* Cyan - 'c'
* Magenta - 'm'
* Yellow - 'y'
* Black - 'k'
* White  - 'w'

Built-in line and marker options:

* Solid Line - '-'
* Dashed - '-'
* Dotted - '.'
* Dash-dotted - '-:'
* Point - '.'
* Pixel - ','
* Circle - 'o'
* Square - 's'
* Triangle - '^'



Subplots
--------

Using the subplot() function, we can plot two graphs at the same time within the same "canvas".
Think of the subplots as "tables", each subplot is set with the number of rows, the number of columns, 
and the active area, the active areas are numbered left to right, then up to down.

.. code-block:: python3

   plt.subplot(2, 1, 1) # (row, column, active area)
   plt.plot(x, np.sin(x))
   plt.subplot(2, 1, 2) # switch the active area
   plt.plot(x, np.sin(2*x))
   plt.savefig('my_sinwavex2b.png')
   plt.show()
    
Scatter plots
-------------

A simple scatter plot based on the sine function:

.. code-block:: python3

   y = np.sin(x)
   plt.scatter(x,y)
   plt.savefig('my_scattersin.png')
   plt.show()
    
Use random numbers and add a colormap to a scatter plot:

.. code-block:: python3

   x = np.random.rand(1000)
   y = np.random.rand(1000)
   size = np.random.rand(1000) * 50
   color = np.random.rand(1000)
   plt.scatter(x, y, size, color)
   plt.colorbar()
   plt.savefig('my_scatterrandom.png')
   plt.show()
    
We brought in two new parameters, size and color, which will vary the diameter and the 
color of our points. Then adding the colorbar() gives us a nice color legend to the side.


Histograms
----------

A histogram is one of the simplest types of graphs to plot in Matplotlib. All you need to do is pass the hist() 
function an array of data. The second argument specifies the amount of bins to use. Bins are intervals of values 
that our data will fall into. The more bins, the more bars.

.. code-block:: python3

   plt.hist(x, 50)
   plt.savefig('my_histrandom.png')
   plt.show()
    
Adding Labels and Legends
-------------------------

Plots look much more finished and professional with appropriate labels and
legends added. This is highly recommended for the final project.

.. code-block:: python3

   x = np.linspace(0, 2 * np.pi, 50)
   plt.plot(x, np.sin(x), 'r-x', label='Sin(x)')
   plt.plot(x, np.cos(x), 'g-^', label='Cos(x)')
   plt.legend() # Display the legend.
   plt.xlabel('Rads') # Add a label to the x-axis.
   plt.ylabel('Amplitude') # Add a label to the y-axis.
   plt.title('Sin and Cos Waves') # Add a graph title.
   plt.savefig('my_labels_legends.png')
   plt.show()


EXERCISE
~~~~~~~~

For this exercise, take your existing Worker app and implement some of the matplotlib
code above to illustrate the results of a job.


Additional Resources
--------------------

* `Try Jupyter in a Browser <https://jupyter.org/>`_
* `Set up X-Forwarding <https://kb.iu.edu/d/bdnt>`_
* `Post Images to Imgur <https://apidocs.imgur.com/>`_
