Storing Images in Redis
=======================

As part of the final project, your Worker must create an image or a plot. If it 
is created inside the Kubernetes worker pod, you'll need a convenient way to 
retrieve that image back out of the worker container and into whatever container
you curled from.

The easiest way to retrieve the image is for the worker to add the image back
to the Redis db, and for the user to query the database with a flask route and
retrieve the image. This would be the general workflow:


1. The user submits a curl request from, e.g., the py-debug pod to the Flask API
2. The Flask API creates a new job entry in the appropriate Redis db, and adds the UUID to the queue
3. The Worker picks up the UUID from the queue, and pulls the job parameters of the Redis db
4. The Worker performs the job task and generates an image or plot
5. The Worker saves the plot in the Redis db, associated with the UUID
6. The user curls a new route to download the image from the db



Initiate a Job
--------------

Imagine that when a user submits a job, an entry is created in the jobs db of the 
following form:

.. code-block:: console

   [user-vm]$ curl localhost:5000/submit -X POST \
                                         -d '{"start": "2001", "end": "2021"}' \
                                         -H "Content-Type: application/json" 
   Job 161207aa-9fe7-4caa-95b8-27f5bcbb16e7 successfully submitted


.. code-block:: console

   [user-vm]$ curl localhost:5000/jobs
   [
     "161207aa-9fe7-4caa-95b8-27f5bcbb16e7"
   ]

.. code-block:: console

   [user-vm]$ curl localhost:5000/jobs/161207aa-9fe7-4caa-95b8-27f5bcbb16e7
   {
     "jobID": "161207aa-9fe7-4caa-95b8-27f5bcbb16e7",
     "status": "submitted",
     "start": "2001",
     "end": "2021"
   }

Add an Image to a Redis DB
--------------------------

The Worker picks up the job, performs the analysis based on the 'start' and
'end' dates, and generates a plot. An example of using ``matplotlib`` to write
and save a plot to file might look like:


.. code-block:: python3
   :linenos:

   import matplotlib.pyplot as plt

   x_values_to_plot = []
   y_values_to_plot = []

   for key in raw_data.keys():       # 'raw_data' is a client to the raw data stored in redis
       if (int(start) <= key['date'] <= int(end)):
           x_values_to_plot.append(key['interesting_property_1'])
           y_values_to_plot.append(key['interesting_property_2'])

   plt.scatter(x_values_to_plot, y_values_to_plot)
   plt.savefig('/output_image.png')
    

.. warning::

   The code above should be considered pseudo code and not copy/pasted directly.
   Depending on how your databases are set up, client names will probably be 
   different, you may need to decode values, and you may need to cast type on
   values.

.. warning::

   Also not shown above is the ``@q`` decorator. The above lines of code should 
   probably be put in their own function, which would be called from the decorated
   function.


Now that an image has been generated, consider the following code that will open up
the image and add it to the Redis db:

.. code-block:: python3

    with open('/output_image.png', 'rb') as f:
        img = f.read()

    results.hset(jobid, 'image', img)       # 'results' is a client to the results db


Retrieve the Image with a Flask Route
-------------------------------------

Now that the image has been added back to the database, you can expect this
type of data structure to exist:


.. code-block:: console

   {
     "161207aa-9fe7-4caa-95b8-27f5bcbb16e7": {
       "image": <binary image data>
     }
   }

It would not be a good idea to show that binary image data with the rest of the
text output when querying a ``/jobs`` route - it would look like a bunch of
random characters. Rather, write a new route to download just the image given the
job ID:

.. code-block:: python3

   from flask import Flask, request, send_file

   @app.route('/download/<jobid>', methods=['GET'])
   def download(jobid):
       path = f'/app/{jobid}.png'
       with open(path, 'wb') as f:
           f.write(results.hget(jobid, 'image'))   # 'results' is a client to the results db
       return send_file(path, mimetype='image/png', as_attachment=True)


Flask has a method called 'send_file' which can return a local file, in this
case meaning a file that is saved inside the Flask container. So first, open
a file handle to save the image file inside the Flask container, then return
the image as ``mimetype='image/png'``.

The setup above will print the binary code to the console, so the user should 
redirect the output to file like:

.. code-block:: console

   [user-vm]$ curl localhost:5000/download/161207aa-9fe7-4caa-95b8-27f5bcbb16e7 --output output.png
   [user-vm]$ ls
   output.png

