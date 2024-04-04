Desired State with Deployments
==============================

In this module, we discuss the Kubernetes abstraction called "deployments". After working through this 
module, students should be able to:

* Explain the types of workloads that should be scheduled using the deployment abstraction in Kubernetes
* Describe a deployment in a yaml file and schedule the deployment on a Kubernetes cluster using ``kubectl``
* Scale the pods associated with a deployment
* Define an image naming and tagging scheme to manage the development and deployment lifecycle of an application

Introduction to Deployments
---------------------------

Deployments are an abstraction and resource type in Kubernetes that can be used to represent long-running application
components, such as databases, REST APIs, web servers, or asynchronous worker programs. The key idea with deployments is
that they should *always be running*.


Imagine a program that runs a web server for a blog site. The blog website should always be available, 24 hours a day,
7 days a week. If the blog web server program crashes, it would ideally be restarted immediately so that the blog site
becomes available again. This is the main idea behind deployments.

Deployments are defined with a pod definition and a replication strategy, such as, "run 3 instances of this pod across
the cluster" or "run an instance of this pod on every worker node in the k8s cluster."

For this class, we will define deployments for our Flask application and its associated components, as deployments
come with a number of advantages over defining "raw" pods. Deployments:

* Can be used to run multiple instances of a pod, to allow for more computing to meet demands put on a system.
* Are actively monitored by k8s for health -- if a pod in a deployment crashes or is otherwise deemed unhealthy, k8s
  will try to start a new one automatically.


Creating a Basic Deployment
---------------------------

We will use yaml to describe a deployment just like we did in the case of pods. Copy and paste the following into a file
called ``deployment-basic.yml``

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-deployment
     labels:
       app: hello-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: hello-app
     template:
       metadata:
         labels:
           app: hello-app
       spec:
         containers:
           - name: hellos
             image: ubuntu:22.04
             command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']


Let's break this down. Recall that the top four attributes are common to all k8s resource descriptions, however it is
worth noting:

* ``apiVersion`` -- We need to use version ``apps/v1`` here. In k8s, different functionalities are packaged into
  different APIs. Deployments are part of the ``apps/v1`` API, so we must specify that here.
* ``metadata`` -- The ``metadata.name`` gives our deployment object a name. This part is similar to when we defined pods.
  We are also using ``labels``. Recall that k8s uses labels to allow objects to refer to other objects in a decoupled way.
  A label in k8s is nothing more than a ``name: value`` pair that users create to organize objects and add information
  meaningful to the user. In this case, ``app`` is the name and ``hello-app`` is the value. Conceptually, you can think
  of label names like variables and labels values as the value for the variable. In some other deployment, we may choose
  to use label ``app: profiles`` to indicate that the deployment is for the "profiles" app.

Let's look at the ``spec`` stanza for the deployment above.

* ``replicas`` -- Defines how many pods we want running at a time for this deployment, in this case, we are asking
  that just 1 pod be running at a time.
* ``selector`` -- This is how we tell k8s where to find the pods to manage for the deployment. Note we are using labels
  again here, the ``app: hello-app`` label in particular.
* ``template`` -- Deployments match one or more pod descriptions defined in the template. Note that in the ``metadata``
  of the template, we provide the same label (``app: hello-app``) as we did in the ``matchLabels`` stanza of the
  ``selector``. This tells k8s that this spec is part of the deployment.
* ``template.spec`` -- This is a pod spec, just like we worked with last time.

.. note::

   If the labels, selectors and matchLables seems confusing and complicated, that's understandable. These semantics allow
   for complex deployments that dynamically match different pods, but for the deployments in this class, you will not
   need this extra complexity. As long as you ensure the label in the ``template`` is the same as the label in the
   ``selector.matchLables`` your deployments will work. It's worth pointing out that the first use of the ``app: hello-app``
   label for the deployment itself (lines 5 and 6 of the yaml) could be removed without impacting the end result.


We create a deployment in k8s using the ``apply`` command, just like when creating a pod:

.. code-block:: console

   [user-vm]$ kubectl apply -f deployment-basic.yml

If all went well, k8s response should look like:

.. code-block:: console

   deployment.apps/hello-deployment created

We can list deployments, just like we listed pods:

.. code-block:: console

   [user-vm]$ kubectl get deployments
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   hello-deployment   1/1     1            1           1m

We can also list pods, and here we see that k8s has created a pod for our deployment for us:

.. code-block:: console

   [user-vm]$ kubectl get pods
   NAME                               READY   STATUS    RESTARTS   AGE
   hello                              1/1     Running   0          10m
   hello-deployment-55c5b77fc-hqjwx   1/1     Running   0          50s
   hello-label                        1/1     Running   0          4m54s

Note that we see our "hello" and "hello-label" pods from earlier as well as a new pod, 
"hello-deployment-9794b4889-kms7p", that k8s created for our deployment. We can use all the kubectl 
commands associated with pods, including listing, describing and
getting the logs. In particular, the logs for our "hello-deployment-9794b4889-kms7p" pod prints the 
same "Hello, Kubernetes!" message, just as was the case with our first pod.


Deleting Pods
-------------

However, there is a fundamental difference between the "hello" pod we created before and our "hello" deployment which
we have alluded to. This difference can be seen when we delete pods.

To delete a pod, we use the ``kubectl delete pods <pod_name>`` command. Let's first delete our hello deployment pod:

.. code-block:: console

   [user-vm]$ kubectl delete pods hello-deployment-55c5b77fc-hqjwx

It might take a little while for the response to come back, but when it does you should see:

.. code-block:: console

   pod "hello-deployment-55c5b77fc-hqjwx" deleted

If we then immediately list the pods, we see something interesting:

.. code-block:: console

   [user-vm]$ kubectl get pods
   NAME                               READY   STATUS    RESTARTS   AGE
   hello                              1/1     Running   0          13m
   hello-deployment-55c5b77fc-76lzz   1/1     Running   0          39s
   hello-label                        1/1     Running   0          7m25s

We see a new pod (in this case, "hello-deployment-55c5b77fc-76lzz") was created and started by k8s for our hello
deployment automatically! k8s did this because we instructed it that we wanted 1 replica pod to be running in the
deployment's ``spec`` -- this was the *desired* state -- and when that didn't match the actual state (0 pods)
k8s worked to change it. Remember, deployments are for programs that should *always be running*.

What do you expect to happen if we delete the original "hello" pod? Will k8s start a new one? Let's try it

.. code-block:: console

   [user-vm]$ kubectl delete pods hello
   pod "hello" deleted

   [user-vm]$ kubectl get pods
   NAME                               READY   STATUS    RESTARTS   AGE
   hello-deployment-55c5b77fc-76lzz   1/1     Running   0          19m
   hello-label                        1/1     Running   0          26m

k8s did not start a new one. This "automatic self-healing" is one of the major difference between deployments and pods.




Scaling a Deployment
--------------------

If we want to change the number of pods k8s runs for our deployment, we simply update the ``replicas`` attribute in
our deployment file and apply the changes. Let's modify our "hello" deployment to run 4 pods. Modify
``deployment-basic.yml`` as follows:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 9

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-deployment
     labels:
       app: hello-app
   spec:
     replicas: 4
     selector:
       matchLabels:
         app: hello-app
     template:
       metadata:
         labels:
           app: hello-app
       spec:
         containers:
           - name: hellos
             image: ubuntu:22.04
             command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Apply the changes with:

.. code-block:: console

   [user-vm]$ kubectl apply -f deployment-basic.yml
   deployment.apps/hello-deployment configured

When we list pods, we see k8s has quickly implemented our requested change:

.. code-block:: console

   [user-vm]$ kubectl get pods
   NAME                               READY   STATUS    RESTARTS   AGE
   hello-deployment-55c5b77fc-76lzz   1/1     Running   0          22m
   hello-deployment-55c5b77fc-nsx6w   1/1     Running   0          9s
   hello-deployment-55c5b77fc-wt4fz   1/1     Running   0          9s
   hello-deployment-55c5b77fc-xtfb9   1/1     Running   0          9s
   hello-label                        1/1     Running   0          29m


EXERCISE
~~~~~~~~

1) Delete several of the hello deployment pods and see what happens.
2) Scale the number of pods associated with the hello deployment back down to 1.


Updating Deployments with New Images
------------------------------------

When we have made changes to the software or other aspects of a container image and we are ready to deploy the new
version to k8s, we have to update the pods making up the corresponding deployment. We will use two different strategies,
one for our "test" environment and one for "production".

Test Environments
~~~~~~~~~~~~~~~~~

A standard practice in software engineering is to maintain one or more "pre-production" environments, often times called
"test" or "quality assurance" environments. These environments look similar to the "real" production environment where
actual users will interact with the software, but few if any real users have access to them. The idea is that software
developers can deploy new changes to a test environment and see if they work without the risk of potentially breaking
the software for real users if they encounter unexpected issues.

Test environments are essential to maintaining quality software, and every major software project the Cloud and
Interactive Computing group at TACC develops makes use of multiple test environments. We will have you create separate
test and production environments as part of building the final project in this class.

It is also common practice to deploy changes to the test environment often, as soon as code is ready and tests are passing
on a developer's laptop. We deploy changes to our test environments dozens of times a day while a large enterprise like
Google may deploy many thousands of times a day. We will learn more about test environments and automated deployment strategies
in the Continuous Integration section.


Image Management and Tagging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you have seen, the ``tag`` associated with a Docker image is the string after the ``:`` in the name. For example,
``ubuntu:22.04`` has a tag of ``22.04`` representing the version of Ubuntu packaged in the image, while
``username/hello-flask:dev`` has a tag of ``dev``, in this case indicating that the image was built from the ``dev`` branch
of the corresponding git repository. Use of tags should be deliberate and is an important detail in a well designed
software development release cycle.

Once you have created a deployment for a pod with a given image,
there are two basic approaches to deploying an updated version of the container images to k8s:

1. Use a new image tag or
2. Use the same image tag and instruct k8s to download the image again.

Using new tags is useful and important whenever you may want to be able to recover or revert back to the previous 
image easily, but on the other hand, it can be tedious to update the tag every time there is a minor 
change to a software image.

Therefore, we suggest the following guidelines for image tagging:

1. During development when rapidly iterating and making frequent deployments, use a tag such as ``dev`` to indicate the
   image represents a development version of the software (and is not suitable for production) and simply overwrite the
   image tag with new changes. Instruct k8s to always try to download a new version of this tag whenever it creates a
   pod for the given deployment (see next section).
2. Once the primary development has completed and the code is ready for end-to-end testing and evaluation, begin to use
   new tags for each change.  These are sometimes called "release candidates" and therefore, a tagging scheme such as
   ``rc1``, ``rc2``, ``rc3``, etc., can be used for tagging each release candidate.
3. Once testing has completed and the software is ready to be deployed to production, tag the image with the version of
   the software. There are a number of different schemes for versioning software, such as Semantic Versioning (https://semver.org/),
   which has been mentioned previously.


ImagePullPolicy
~~~~~~~~~~~~~~~

When defining a deployment, we can specify an ``ImagePullPolicy`` which instructs k8s about when and how to download
the image associated with the pod definition. For our test environments, we will instruct k8s to always try and
download a new version of the image whenever it creates a new pod. We do this by specifying ``imagePullPolicy: Always``
in our deployment.

For example, we can add ``imagePullPolicy: Always`` to our hello-deployment as follows:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 20

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-deployment
     labels:
       app: hello-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: hello-app
     template:
       metadata:
         labels:
           app: hello-app
       spec:
         containers:
           - name: hellos
             imagePullPolicy: Always
             image: ubuntu:22.04
             command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Now k8s will always try to download the latest version of ``ubuntu:22.04`` from Docker Hub every time it creates
a new pod for this deployment. As discussed above, using ``imagePullPolicy: Always`` is nice during active development
because you ensure k8s is always deploying the latest version of your code. Other possible values include
``IfNotPresent`` (the current default) which instructs k8s to only pull the image if it doesn't already exist on the
worker node. This is the proper setting for a production deployment in most cases.


Deleting Pods to Update the Deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that if we have an update to our ``:dev`` image and we have set ``imagePullPolicy: Always`` on our deployment, all
we have to do is delete the existing pods in the deployment to get the updated version deployed: as soon as we delete the
pods, k8s will determine that an insufficient number of pods are running and try to start new ones. The ``imagePullPolicy``
instructs k8s to first try and download a newer version of the image.

.. note::

   Consult the software diagram to understand the flow of source code changes => updated container running on the 
   k8s cluster.




Additional Resources
--------------------

 * `Kubernetes Deployments Documentation <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>`_




