Flask Special Topics
====================


Writing Unit Tests for Flask
----------------------------

There are several different approaches for writing *unit tests* for Flask apps,
some of which are included in the Flask documentation. The approach I like the
best, and which is most congruent with the topics we already covered in this class,
is to instead write *integration tests*. 

For these tests, the key thing is that you have to have an actual copy of your
Flask server running. Then, the tests will perform requests.get() to the various
routes and you can evaluate the output. Be careful to make sure that your tests
are independent of the data, which may change over time. The job of these unit
tests is not to check / validate that the right data is being returned; rather,
the goal is to make sure the routes work and return the right kind of response.

If your code also has standalone functions separate from the functions decorated
as Flask routes, (e.g., in the case of the ISS data you may have written
standalone functions for finding the time closest to now or computing average
speed that are called from other functions), then you would still want to import
those and test them as normal in your unit test file.

Consider the following pytest snippet that tests the /epochs and /epochs/<epoch>
routes for the ISS tracker:


.. code-block:: python3
   :linenos:

   import pytest
   import requests

   response1 = requests.get('http://127.0.0.1:5000/epochs')
   a_representative_epoch = response1.json()[0]
   response2 = requests.get('http://127.0.0.1:5000/epochs/'+a_representative_epoch)

   def test_epochs_route():
       assert response1.status_code == 200
       assert isinstance(response1.json(), list) == True

   def test_specific_epoch_route():
       assert response2.status_code == 200
       assert isinstance(response2.json(), dict) == True



These lines may need to be modified depending on the format of data returned by
your API. And, as mentioned above, it assumes these tests are performed when your
API is already running.


Reverse Proxy with ``ngrok``
----------------------------

Up to now, our Flask apps have been isolated to their host hardware. What if 
we want to access our Flask apps from outside our host hardware? A utility
called ``ngrok`` can be used to provide a reverse proxy on the fly.

.. note::
 
   To do this, you need an ngrok account and access token, which can be
   obtained `here <https://dashboard.ngrok.com/signup>`_

First, install ``ngrok``:

.. code-block:: console

   [user-vm]$ curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | \
                   sudo gpg --dearmor -o /etc/apt/keyrings/ngrok.gpg && \
                   echo "deb [signed-by=/etc/apt/keyrings/ngrok.gpg] https://ngrok-agent.s3.amazonaws.com buster main" | \
                   sudo tee /etc/apt/sources.list.d/ngrok.list && \
                   sudo apt update && sudo apt install ngrok

Confirm ``ngrok`` is working and add your
`access token <https://dashboard.ngrok.com/get-started/your-authtoken>`_:

.. code-block:: console

   [user-vm]$ ngrok --help
   NAME:
     ngrok - tunnel local ports to public URLs and inspect traffic
   ...
   [user-vm]$ ngrok config add-authtoken <TOKEN>
   Authtoken saved to configuration file: /home/ubuntu/.config/ngrok/ngrok.yml


Assuming you have a Flask app running on ``localhost`` and port ``5000``, then
do the following: 

.. code-block:: console

   [user-vm]$ ngrok http http://localhost:5000


This will lock your terminal into an interface that looks like the following:

.. code-block:: console

   ngrok
   
   K8s Gateway API https://ngrok.com/early-access/kubernetes-gateway-api
   
   Session Status                online
   Account                       username (Plan: Free)
   Version                       3.8.0
   Region                        United States (us)
   Latency                       37ms
   Web Interface                 http://127.0.0.1:4040
   Forwarding                    https://9750-129-141-63-209.ngrok-free.app -> http://localhost:5000
   
   Connections                   ttl     opn     rt1     rt5     p50     p90

Navigate to or curl the link provided to access your Flask app from outside
the Jetstream VM. Press ``Ctrl+C`` to quit forwarding.


Flask and HTML
--------------

Flask has the ability to render HTML templates that contain a mix of static data
and variables (for run-time dynamic data). The route you write will take the 
template from a predefined location, inject any variables, and render it into
a final HTML document to return to the client.

In your Flask project directory, create a folder called ``templates`` and add
the following HTML document as ``example.html``. It uses Jinja syntax for
injecting dynamic data (``{{name}}``):

.. code-block:: html

   <!DOCTYPE html>
   <html>
   <head>
       <title>Flask Templating Example</title>
   </head>
   <body>
       <h1>Hello, {{name}}!</h1>
       <p>This file should be stored in the "templates" folder as "example.html"</p>
   </body>
   </html>

Then, adapt the ``/<name>`` Flask route to return that HTML document using Flask's 
``render_template()`` method. The old version of the route which returns a plain
string is also provided as reference. 

.. code-block:: python3
   :linenos:

   from flask import Flask, render_template
   
   app = Flask(__name__)
   
   @app.route('/<name>', methods=['GET'])
   def hello_name(name):
       return render_template('example.html', name=name)

   ### For reference, this is the /<name> route without render_template
   # @app.route('/<name>', methods=['GET'])
   # def hello_name(name):
   #     return f'Hello, {name}!\n'
   
   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')

   



Additional Resources
--------------------

* `Ngrok <https://dashboard.ngrok.com/signup>`_
* `Flask Templates <https://flask.palletsprojects.com/en/2.3.x/tutorial/templates/>`_
