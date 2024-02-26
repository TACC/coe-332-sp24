Containerizing Flask
====================

As we have discussed previously, Docker containers are critical to packaging an
application along with all of its dependencies, isolating it from
other applications and services, and deploying it in a consistent and reproducible
way across different platforms.

Here, we will walk through the process of containerizing a Flask application with Docker, and then
using ``curl`` to interact with it as a containerized microservice. After going through this
module, students should be able to:

* Assemble the different components needed for a containerized microservice into on directory.
* Establish and document requirements (e.g. dependencies, Python packages) for the project.
* Build and run in the background a containerized Flask microservice.
* Map ports on the Jetstream VM to ports inside a container, and use ``curl`` with the
  the correct ports to make requests to and generate responses from the microservice.
* Deploy the microservice with docker-compose
* **Design Principles:** By combining Flask and Docker, we will see how both contribute to
  the *modularity*, *portability*, *abstraction*, and *generalization* of software (all
  four major design principles).



Organize Your App Directory
---------------------------


First, create a new directory for this exercise, and change directories to it:

.. code-block:: console

   [user-vm]$ mkdir flask-container && cd flask-container
   [user-vm]$ pwd
   /home/ubuntu/flask-container

Then, create a new ``app.py`` (or copy an existing one) into this folder. It
should have the following contents:

.. code-block:: python3
   :linenos:

   from flask import Flask

   app = Flask(__name__)

   @app.route('/', methods = ['GET'])
   def hello_world():
       return 'Hello, world!\n'

   @app.route('/<name>', methods = ['GET'])
   def hello_name(name):
       return f'Hello, {name}!\n'

   if __name__ == '__main__':
       app.run(debug=True, host='0.0.0.0')


Specify Requirements
---------------------

The Python package manager ``pip`` can utilize a text file for managing package
dependencies of your application. It is standard practice to to capture the required
libraries and packages for a project in a file called ``requirements.txt``. For
our example here, create a file called ``requirements.txt`` and add the following line:

.. code-block:: console

   Flask==3.0.2

This indicates that our project requires the ``Flask`` package, version number ``3.0.2``.
You can specify your requirements in more lenient ways -- for example, we could have
put ``Flask>=3.0.2`` to indicate that any version greater than or equal to ``3.0.2`` would work,
or we could have even put ``Flask`` with no version indicating we don't care what version of
Flask is installed.

Note:

* Specifying a package, such as ``Flask`` as a dependency instructs pip to install Flask *and all
  of its dependencies*. Those dependencies could in turn have dependencies, etc., and pip will
  take care of installing all of those.
* Specifying the exact version improves the odds that your application will work correctly because
  the packages that get installed will be the versions you specified. Therefore, it is
  usually best to specify the exact version of the library your application requires.


Build a Docker Image
--------------------

As we saw in a previous section, we write up the recipe for our application
installation process in a Dockerfile. Create a file called ``Dockerfile`` for our
Flask microservice and add the following lines:

.. code-block:: dockerfile
   :linenos:

    FROM python:3.9

    RUN mkdir /app
    WORKDIR /app
    COPY requirements.txt /app/requirements.txt
    RUN pip install -r /app/requirements.txt
    COPY app.py /app/app.py

    ENTRYPOINT ["python"]
    CMD ["app.py"]


Here we see usage of the Docker ``ENTRYPOINT`` and ``RUN`` instructions, which
essentially specify a default command (``python app.py``) that should be run
when an instance of this image is instantiated.

Note also that we copied the ``requirements.txt`` file before copying the full
current working directory. Why did we do that?

The answer has to do with how Docker caches image layers. We could have written the following
instead:

.. code-block:: dockerfile
   :linenos:

    FROM python:3.9

    RUN mkdir /app
    WORKDIR /app
    COPY . /app
    RUN pip install -r /app/requirements.txt

    ENTRYPOINT ["python"]
    CMD ["app.py"]

The above is actually shorter; i.e., fewer lines of code in the Dockerfile.

However, with the above approach, Docker is going to re-run the command
``pip install -r /app/requirements.txt`` every time there is any change to the contents
of the current working directory (i.e., any time we change our app code or any other files).
This is not a big deal with a small ``requirements.txt`` file and only a few packages to install,
but as the ``requirements.txt`` file gets bigger, the time to install all the packages
can be significant.

As a general rule of thumb, put more expensive (in term of time) operations whose are less likely
to change at the beginning of your ``Dockerfile`` to maximize the value of the Docker image
layer cache.


Save the file and build the image with the following command:

.. code-block:: console

   [user-vm]$ docker build -t username/flask-helloworld:1.0 .

.. warning:

   Don't forget to replace ``<username>`` with your Docker Hub username.

Run a Docker Container
----------------------

To create an instance of your image (a "container"), use the following command:

.. code-block:: console

   [user-vm]$ docker run --name "flask-helloworld-app" -d -p 5000:5000 username/flask-helloworld:1.0

The ``-d`` flag detaches your terminal from the running container - i.e. it
runs the container in the background. The ``-p`` flag maps a port on the Jetstream
VM (5000, in the above case) to a port inside the container (again 5000, in the
above case). In the above example, the Flask app was set up to use the
default port inside the container (5000), and we can access that through our
specified port on Jetstream (5000). This explicit mapping is convenient if you 
have multiple services running on the same VM and you want to avoid port
collisions. 

Check to see that things are up and running with:

.. code-block:: console

   [user-vm]$ docker ps -a

The list should have a container with the name you gave it, an ``UP`` status,
and the port mapping that you specified.

If the above is not found in the list of running containers, try to debug with
the following:

.. code-block:: console

   [user-vm]$ docker logs "your-container-name"
   -or-
   [user-vm]$ docker logs "your-container-ID"


Access Your Microservice
------------------------

Now for the payoff - you can use ``curl`` to interact with your Flask microservice by specifying
the correct port on the ISP server. Following the example above, which was using
port 5000:

.. code-block:: console

   [user-vm]$ curl localhost:5000/
   Hello, world!
   [user-vm]$ curl localhost:5000/Joe
   Hello, Joe!


Clean Up
--------

Finally, don't forget to stop your running container and remove it.

.. code-block:: console

   CONTAINER ID   IMAGE                           COMMAND           CREATED         STATUS         PORTS                                       NAMES
   a785237628d6   username/flask-helloworld:1.0   "python app.py"   4 minutes ago   Up 4 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   flask-helloworld-app
   [user-vm]$ docker stop a785237628d6
   a785237628d6
   [user-vm]$ docker rm a785237628d6
   a785237628d6


EXERCISE
~~~~~~~~

Containerize your Flask degrees app from last week:

1. Create a Dockerfile for your app
2. Build the image from the Dockerfile
3. Run the server locally and test the endpoints using curl



Docker Compose, Revisited
-------------------------

Using the ``docker run`` command to start containers is OK for simple commands, but as 
we started to see in the previous material, the commands can get long pretty quickly. It can be
hard to remember all of the flags and options that we want to use when starting our
containers. 

Moreover, so far we have been looking at single-container applications. 
But what if we want to do something more complex involving multiple containers? In this course, 
our goal is to ultimately develop and orchestrate a multi-container
application consisting of, e.g., a Flask app, a database, a message queue, an
authentication service, and more.


Write a Compose File
--------------------

Docker compose works by interpreting rules declared in a YAML file (typically
called ``docker-compose.yml``). The rules we will write will replace the
``docker run`` commands we have been using, and which have been growing quite
complex. Recall from the past exercise that the command we were using to start our Flask 
application container looked like the following:

.. code-block:: console

   [user-vm]$ docker run --name "flask-helloworld-app" -d -p 5000:5000 username/flask-helloworld:1.0

The above ``docker run`` command can be translated into a YAML file.
Navigate to the folder that contains your Python scripts and Dockerfiles, then
create a new empty file called ``docker-compose.yml``:

.. code-block:: console

   [user-vm]$ pwd
   /home/ubuntu/flask-contaienr
   [user-vm]$ touch docker-compose.yml
   [user-vm]$ ls
   Dockerfile  app.py  docker-compose.yaml  requirements.txt


Next, open up ``docker-compose.yml`` with your favorite text editor and type /
paste in the following text:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 9

   ---
   version: "3"

   services:
       flask-app:
           build:
               context: ./
               dockerfile: ./Dockerfile
           image: username/flask-helloworld:1.0
           container_name: flask-helloworld-app
           ports:
               - "5000:5000"
   ...

.. note::

   Be sure to update the highlighted line above with your username.


The ``version`` key must be included and simply denotes that we are using
version 3 of Docker compose.

The ``services`` section defines the configuration of individual container
instances that we want to orchestrate. In our case, we define just one container
called ``flask-app``. We can use any allowable name for the services we defined, but each
name should be unique within the docker-compose.yml file. 

The ``flask-app`` service is configured with its own Docker image, including a
reference to a Dockerfile to be used to ``build`` the image, a recognizable name
for the running container, and a port mapping for the Flask service. Recall from
the `previous unit <../unit05/docker_compose.html>`_ that other speicifcations
can be defined in this file including a list of mounted volumes, user IDs for
running the service, default commands, and many others. The choice of which 
options to use entirely depends on the app and the context.

.. note::

   The top-level ``services`` keyword shown above is just one important part of
   Docker compose. Later in this course we will look at named volumes and
   networks which can be configured and created with Docker compose.


Running Docker Compose
----------------------

To run our Flask application container, we simply use the ``docker-compose up`` 
verb, which will start up all containers defined in the file. Alternatively,
we could use ``docker-compose run`` and pass the name of a service to run, in this
case, ``flask-app``:

.. code-block:: console

   [user-vm]$ docker-compose up 
   Creating network "flask-container_default" with the default driver
   Creating flask-helloworld-app ... done
   Attaching to flask-helloworld-app
   flask-helloworld-app |  * Serving Flask app 'app'
   flask-helloworld-app |  * Debug mode: on
   flask-helloworld-app | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
   flask-helloworld-app |  * Running on all addresses (0.0.0.0)
   flask-helloworld-app |  * Running on http://127.0.0.1:5000
   flask-helloworld-app |  * Running on http://172.23.0.2:5000
   flask-helloworld-app | Press CTRL+C to quit
   flask-helloworld-app |  * Restarting with stat
   flask-helloworld-app |  * Debugger is active!
   flask-helloworld-app |  * Debugger PIN: 109-459-387

Note that ``docker-compose`` starts the container in the foreground and takes over our terminal. If we use 
``Ctrl+C`` we will stop the container. We can see confirm that the container is stopped using the
``docker ps -a`` command:

.. code-block:: console

   [user-vm] docker ps -a 
   CONTAINER ID   IMAGE                           COMMAND           CREATED          STATUS                     PORTS     NAMES
   289ea2d0fed6   username/flask-helloworld:1.0   "python app.py"   32 seconds ago   Exited (0) 4 seconds ago             flask-helloworld-app


To start the service in the background, use the ``-d`` flag:

.. code-block:: console

   [user-vm]$ docker-compose up -d

Once the service is running, perform some curl commands to test the running Flask
app before stopping the service with:


.. code-block:: console

   [user-vm]$ docker-compose down



Essential Docker Compose Command Summary
----------------------------------------

+------------------------+------------------------------------------------+
| Command                | Usage                                          |
+========================+================================================+
| docker-compose version | Print version information                      |
+------------------------+------------------------------------------------+
| docker-compose config  | Validate docker-compose.yml syntax             |
+------------------------+------------------------------------------------+
| docker-compose up      | Spin up all services                           |
+------------------------+------------------------------------------------+
| docker-compose down    | Tear down all services                         |
+------------------------+------------------------------------------------+
| docker-compose build   | Build the images listed in the YAML file       |
+------------------------+------------------------------------------------+
| docker-compose run     | Run a container as defined in the YAML file    |
+------------------------+------------------------------------------------+


Additional Resources
--------------------

* `Docker Compose Docs <https://docs.docker.com/compose/>`_
