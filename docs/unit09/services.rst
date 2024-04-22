Services and Ingress
====================

Services are the k8s resource one uses to expose HTTP APIs, databases and other components that communicate
on a network to other k8s pods and, ultimately, to the outside world. 
After working through this module, students should be able to:

* Set up port forwarding from pods on a Kubernetes cluster
* Use a debug deployment to test access
* Create and attach a service to a deployment
* Use a NodePort service to expose a Flask API on the public internet
* Use an Ingress object to map a NodePort port to a subdomain on the host

k8s Networking Overview
-----------------------

To understand services we need to first discuss how networking works in k8s.

.. note::

  We will be covering just the basics of k8s networking, enough for you to become proficient with the main concepts
  involved in deploying your application. Many details and advanced concepts will be omitted.

k8s creates internal networks and attaches pod containers to them to facilitate communication between pods. For a number
of reasons, including security, these networks are not reachable from outside k8s.

Recall that we can learn the private network IP address for a specific pod with the following command:

.. code-block:: console

   [user-vm]$ kubectl get pods <pod_name> -o wide

For example:

.. code-block:: console

   [user-vm]$ kubectl get pods hello-deployment-9794b4889-mk6qw -o wide
   NAME                                    READY   STATUS        RESTARTS       AGE   IP              NODE            NOMINATED NODE   READINESS GATES
   hello-deployment-6949f8ddbc-znx75       1/1     Running       21 (31m ago)   21h   10.233.116.45   kube-worker-1   <none>           <none>



This tells us k8s assigned an IP address of ``10.233.116.45`` to our hello-deployment pod. IP addresses starting with
``10.`` are private IP addresses which are part of a private network, isolated to this k8s cluster.
k8s assigns every pod on this private network an IP address. Pods that are on the same network can communicate with other
pods using their IP address.

Ports
-----
To communicate with a program running on a network, we use ports. We saw how our Flask program used port 5000 to
communicate HTTP requests from clients. We can expose ports in our k8s deployments by defining a ``ports`` stanza in
our ``template.spec.containers`` object. Let's try that now.

Create a file called ``deployment-hello-flask.yml`` and copy the following contents

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-flask
     labels:
       app: hello-flask
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: hello-flask
     template:
       metadata:
         labels:
           app: hello-flask
       spec:
         containers:
           - name: hello-flask
             imagePullPolicy: Always
             image: wjallen/hello-flask:1.0
             ports:
             - name: http
               containerPort: 5000

Much of this will look familiar. We are creating a deployment that matches the pod description given in the ``template.spec``
stanza. The pod description uses an image, ``wjallen/hello-flask:1.0``. This image runs a very simple flask server that
responds with simple text message at the ``/`` endpoint.

The ``ports`` attribute is a list of k8s port descriptions. Each port in the list includes:

  * ``name`` -- the name of the port, in this case, ``http``. This could be anything we want really.
  * ``containerPort`` -- the port inside the container to expose, in this case ``5000``. This needs to match the port
    that the containerized program (in this case, flask server) is binding to.

Next create the hello-flask deployment using ``kubectl apply``

.. code-block:: console

   [user-vm]$ kubectl apply -f deployment-hello-flask.yml
   deployment.apps/deployment-hello-flask configured

With our deployment created, we should see a new pod.

EXERCISE
~~~~~~~~

Determine the IP address of the new pod for the deployment-hello-flask.

SOLUTION
~~~~~~~~


.. code-block:: console

   [user-vm]$ kubectl get pods
   NAME                                READY   STATUS    RESTARTS       AGE
   hello-deployment-6949f8ddbc-znx75   1/1     Running   21 (36m ago)   21h
   hello-label                         1/1     Running   21 (57m ago)   21h
   hello-label2                        1/1     Running   21             21h
   hello-flask-7bf64cc577-l7f52        1/1     Running   0              2m34s


   [user-vm]$ kubectl get pods helloflask-86d4c7d8db-2rkg5 -o wide
   NAME                          READY   STATUS    RESTARTS   AGE     IP              NODE            NOMINATED NODE   READINESS GATES
   hello-flask-7bf64cc577-l7f52  1/1     Running   0          3m46s   10.233.116.59   kube-worker-1   <none>           <none>


  # Therefore, the IP address is 10.233.116.59

We found the IP address for our Flask container, but if we try to communicate with it from our Jetstream VMs, 
we will either find that it hangs indefinitely or possible gives an error:

.. code-block:: console

   [user-vm]$ curl 10.233.116.59:5000/
   curl: (7) Failed connect to 10.233.116.59:5000; Network is unreachable

This is because the 10.233.*.* private k8s network is not available from the outside.
However, it *is* available from other pods in the namespace.


A Debug Deployment
------------------

For exploring and debugging k8s deployments, it can be helpful to have a basic container on the network. We can
create a deployment for this purpose.

For example, let's create a deployment using the official Python 3.10 image. We can run a sleep command inside the
container as the primary command, and then, once the container pod is running, we can use ``exec`` to launch a shell
inside the container.


EXERCISE
~~~~~~~~

Create a new "debug" deployment using the following definition:

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: py-debug
     labels:
       app: py-debug
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: py-debug
     template:
       metadata:
         labels:
           app: py-debug
       spec:
         containers:
           - name: py-debug
             image: python:3.10
             command: ['sleep', '999999999']



Once it is ready, exec into the running pod for this deployment. Once we have a shell running inside our debug
deployment pod, we can try to access our Flask server. Recall that
the IP and port for the Flask server were determined above to be 10.244.7.95:5000 (yours will be different).

If we try to access it using curl from within the debug container, we get:

.. code-block:: console

   root@py-debug-deployment-5cc8cdd65f-xzhzq: $ curl 10.233.116.59:5000
   Hello, world!

Great! k8s networking from within the private network is working as expected!


Services
--------

We saw above how pods can use the IP address of other pods to communicate. However, that is not a great solution because
we know the pods making up a deployment come and go. Each time a pod is destroyed and a new one created it gets a new
IP address. Moreover, we can scale the number of replica pods for a deployment up and down to handle more or less load.

How would an application that needs to communicate with a pod know which IP address to use? If there are 3 pods comprising
a deployment, which one should it use? This problem is referred to as the *service discovery problem* in distributed
systems, and k8s has a solution for it.. the ``Service`` abstraction.

A k8s service provides an abstract way of exposing an application running as a collection of pods on a single IP address
and port. Let's define a service for our hello-flask deployment.


Copy and paste the following code into a file called ``service-hello-flask.yml``:

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hello-flask-service
   spec:
     type: ClusterIP
     selector:
       app: hello-flask
     ports:
     - name: hello-flask
       port: 5000
       targetPort: 5000

Let's look at the ``spec`` description for this service.

  * ``type`` -- There are different types of k8s services. Here we are creating a ``ClusterIP`` service. This creates an
    IP address on the private k8s network for the service. We may see other types of k8s services later.
  * ``selector`` -- This tells k8s what pod containers to match for the service. Here we are using a label,
    ``app: hello-flask``, which means k8s will link all pods with this label to our service. Note that it is important that
    this label match the label applied to our pods in the deployment, so that k8s links the service up to the correct
    pods.
  * ``ports`` - This is a list of ports to expose in the service.
  * ``ports.port`` -- This is the port to expose on the service's IP. This is the port clients will use when communicating
    via the service's IP address.
  * ``ports.targetPort`` -- This is the port on the pods to target. This needs to match the port specified in the pod
    description (and the port the containerized program is binding to).

We create this service using the ``kubectl apply`` command, as usual:

.. code-block:: console

   [user-vm]$ kubectl apply -f hello-flask-service.yml
   service/hello-service configured

We can list the services:

.. code-block:: console

   [user-vm]$ kubectl get services
   NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
   hello-service   ClusterIP   10.233.12.76   <none>        5000/TCP   11s

We see k8s created a new service with IP ``10.233.12.76``. We should be able to use this IP address (and port 5000) to
communicate with our Flask server. Let's try it. Remember that we have to be on the k8s private network, so we need to
exec into our debug deployment pod first.

.. code-block:: console

  [user-vm]$ kubectl exec -it py-debug-deployment-5cc8cdd65f-xzhzq -- /bin/bash

  # from inside the container ---
  root@py-debug-deployment-5cc8cdd65f-xzhzq:/ $ curl 10.233.12.76:5000/
  Hello, world!

It worked! Now, if we remove our hello-flask pod, k8s will start a new one with a new IP address, but our service will
automatically route requests to the new pod. Let's try it.

.. code-block:: bash

   # remove the pod ---
   [user-vm]$ kubectl delete pods hello-flask-86d4c7d8db-2rkg5
   pod "helloflask-86d4c7d8db-2rkg5" deleted

   # see that a new one was created ---
   [user-vm]$ kubectl get pods
   NAME                                    READY   STATUS    RESTARTS   AGE
   hello-deployment-9794b4889-w4jlq        1/1     Running   2          175m
   hello-pvc-deployment-6dbbfdc4b4-sxk78   1/1     Running   233        9d
   hello-flask-86d4c7d8db-vbn4g             1/1     Running   0          62s

   # it has a new IP ---
   [user-vm]$ kubectl get pods helloflask-86d4c7d8db-vbn4g -o wide
   NAME                          READY   STATUS    RESTARTS   AGE    IP            NODE   NOMINATED NODE   READINESS GATES
   hello-flask-86d4c7d8db-vbn4g   1/1     Running   0          112s   10.233.12.96   c05    <none>           <none>
   # Yep, 10.233.12.96 -- that's different; the first pod had IP 10.233.116.59

   # but back in the debug deployment pod, check that we can still use the service IP --
   root@py-debug-deployment-5cc8cdd65f-xzhzq:/ $ curl 10.233.12.76:5000/
   Hello, world!


Note that k8s is doing something non-trivial here. Each pod could be running on one of any number of worker computers in
the TACC k8s cluster. When the first pod was deleted and k8s created the second one, it is quite possible it started it
on a different machine. So k8s had to take care of rerouting requests from the service to the new machine.

k8s can be configured to do this "networking magic" in different ways. While the details are beyond the scope of this
course, keep in mind that the virtual networking that k8s uses does come at a small cost. For most applications,
including long-running web APIs and databases, this cost is negligible and isn't a concern. But for high-performance
applications, and in particular, applications whose performance is bounded by the performance of the underlying network,
the overhead can be significant.



EXERCISE
~~~~~~~~

Now, you have enough tools in your k8s toolbox to deploy your entire web app on the k8s cluster. Follow the steps 
below to try to launch your web app. In each step, you will be creating a new k8s resource described by its own
k8s yaml file. We recommend carefully naming the files following a pattern like ``<name>-<env>-<resource>-<item>.yml``.
In this case ``<name>`` is the name of your app, ``<env>`` is the environment - test or prod, ``<resource>`` is the 
k8s resource used - e.g. deployment or service or pvc, and ``<item>`` is the identity of the service - e.g. flask or
redis or worker. For example, imagine you are creating a deployment for the Flask front end for the test copy of an
web app for analyzing the HGNC data. You might name this yaml file ``HGNC-test-deployment-flask.yml``. Other naming
schemes for these files are perfectly valid as long as there is *consistency* among them.


* **Step 1:** Create a PVC for Redis to write dump files
* **Step 2:** Create a deployment for your Redis database which binds the PVC from Step 1
* **Step 3:** Create a service for your Redis deployment so you have a persistent IP address to use to communicate with Redis
* **Step 4:** Create a deployment for your Flask API 
* **Step 5:** Create a service for your Flask API so you have a persistent IP address to use to communicate with Flask
* **Step 6:** Create a deployment for your Worker which is scaled to three replicas


As you apply files, be sure to check your work in between each step. Make sure deployments and pods are ready and
available, make sure PVCs are bound, make sure services are correctly associated with deployments, etc.
Use a debug deployment to test things as much as possible along the way. 


Public Access to Your Deployment
--------------------------------

The final objective is to make your Flask API available on the public internet.
This process assumes you have already created a deployment and a service (of type
``ClusterIP``) for your Flask API. There are two new k8s objects required:

1. A second ``Service`` object of type ``NodePort`` which selects your deployment using the 
   deployment label and exposes your Flask API on a public port. 
2. An ``Ingress`` object which specifies a subdomain to make your Flask API available on and 
   maps this domain to the public port created in Step 1. 


Create a NodePort Service
--------------------------

The first step is to create a NodePort Service object pointing at your Flask deployment. 
Copy the following code into a new file called ``service-nodeport-hello-flask.yml`` or something 
similar:

.. code-block:: yaml    
   :linenos:
   :emphasize-lines: 5, 9

   ---
   kind: Service
   apiVersion: v1
   metadata:
       name: hello-flask-nodeport-service
   spec:
       type: NodePort
       selector:
           app: hello-flask
       ports:
           - port: 5000
             targetPort: 5000

Update the highlighted lines:

1. The ``name`` of the Service object can be anything you want, so long 
   as it is unique among the Services you have defined in your namespace. In particular, it needs to 
   be a different name from your ClusterIP service defined previously. 

2. The value of ``app`` in the ``selector`` stanza needs to match the ``app`` label in your 
   deployment. This should be exactly the same as what you did in the ClusterIP service created
   previously. As mentioned before, be sure the selector targets the **label** in your Flask
   deployment, **not the deployment name**.
  
As usual, create the NodePort using ``kubectl``:

.. code-block:: console 

   [user-vm]$ kubectl apply -f service-nodeport-hello-flask.yml
   service/hello-flask-nodeport-service created

Change the command to reference the file name you used. 
Check that the service was created successfully and determine the port that was created for it:

.. code-block:: console 

   [user-vm]$ kubectl get services
   NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   hello-flask-nodeport-service   NodePort    10.233.1.87    <none>        5000:32627/TCP   24s
   hello-flask-service            ClusterIP   10.233.58.60   <none>        5000/TCP         4d10h


Here we see that port ``32627`` was created for this service. Your port will be different. 


You can test that the NodePort service is working by using the special domain ``coe332.tacc.cloud``
to exercise your Flask API from the kube-access VM:

.. code-block:: console

    [user-vm]$ curl coe332.tacc.cloud:32627/
    Hello, world!

Change the port (``32627``) to the port associated with your nodeport service, and the URL path
(``/``) to a path your Flask API recognizes. 

.. note::

    The curl above only works on the public internet - in the next section, we will map the 
    port to a subdomain of the host.


Create an Ingress 
-----------------

Next we will create an Ingress object which will map the NodePort port defined previously 
(in the above example, ``32627``) to a specific domain on the public internet. 

Copy the following code into a new file called ``ingress-hello-flask.yml`` or similar:

.. code-block:: yaml    
   :linenos:
   :emphasize-lines: 5, 11, 20

   ---
   kind: Ingress
   apiVersion: networking.k8s.io/v1
   metadata:
     name: hello-flask-ingress
     annotations:
       nginx.ingress.kubernetes.io/ssl-redirect: "false"
   spec:
     ingressClassName: nginx
     rules:
     - host: "username-flask.coe332.tacc.cloud"
       http:
           paths:
           - pathType: Prefix
             path: "/"
             backend:
               service:
                 name: hello-flask-nodeport-service
                 port:
                     number: 5000


Be sure to update the highlighted lines:

1. Specify a meaningful ``name`` for the ingress. Keep in mind it should be unique among all 
   Ingress obejcts within your namespace. 
2. Update the ``host`` value to include your username in the subdomain, i.e., use the format 
   ``- host: "<username>.coe332.tacc.cloud"``.

Create the Ingress object:

.. code-block:: console 

    [user-vm]$ kubectl apply -f ingress-hello-flask.yml


Double check that the object was successfully created:

.. code-block:: console

   [user-vm]$ kubectl get ingress
   NAME                  CLASS    HOSTS                              ADDRESS   PORTS   AGE
   hello-flask-ingress   <none>   username-flask.coe332.tacc.cloud             80      102s

At this point our Flask API should be available on the public internet from the domain 
we specified in the ``host`` field. We can test by running the following curl command from 
anywhere, including our laptops. 


.. code-block:: console

   [local]$ curl username-flask.coe332.tacc.cloud/
   Hello, world!


Additional Resources
--------------------

 * `Services in k8s <https://kubernetes.io/docs/concepts/services-networking/service/>`_

