Introduction to Pods
====================

In this section we give an overview of the first major Kubernetes abstraction, the Pod.

After going through this module, students should be able to:

* Connect to the class Kubernetes cluster and issue basic commands using ``kubectl``
* Describe a Kubernetes pod in a yaml file and use ``kubectl`` to deploy the pod to the cluster
* Retrieve details about the pod including status and logs
* Use labels to organize pods deployed to the Kubernetes cluster
* **Design Principles**. Kubernetes improves our software portability, particularly 
  for large distributed applications that need to run across multiple machines


Introduction to Pods
--------------------

Pods are a fundamental abstraction within Kubernetes and are the most basic unit of computing that can be deployed onto
the cluster. A pod can be thought of as generalizing the notion of a container: a pod contains one or more containers
that are tightly coupled and need to be scheduled together, on the same computer, with access to a shared file system
and a shared network address.

.. note::

  By far, the majority pods you will meet in the wild, including the ones used in this course, will only include one
  container. A pod with multiple containers can be thought of as an "advanced" use case.


Hello, Kubernetes
-----------------

To begin, we will define a pod with one container. As we will do with all the resources we want to create in k8s, we
will describe our pod in a yaml file.

Create a file called ``pod-basic.yml``, open it up in an editor and paste the following code in:

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: hello
   spec:
     containers:
       - name: hello
         image: ubuntu:22.04
         command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Let's break this down. The top four attributes are common to all k8s resource descriptions:

  * ``apiVersion`` -- describes what version of the k8s API we are working in. We are using ``v1``.
  * ``kind`` -- tells k8s what kind of resource we are describing, in this case a ``Pod``.
  * ``metadata`` -- in general, this is additional information about the resource we are describing that doesn't pertain
    to its operation. Here, we are giving our pod a ``name``, ``hello``.
  * ``spec`` -- This is where the actual description of the resource begins. The contents of this stanza vary depending
    on the ``kind`` of resource you are creating. We go into more details on this in the next section.


.. warning::

  Only one Kubernetes object of a specific ``kind`` can have a given ``name`` at a time. If you define a second pod
  with the same name you will overwrite the first pod. This is true of all the different types of k8s objects we will
  be creating.


The Pod Spec
~~~~~~~~~~~~

In k8s, you describe resources you want to create or update using a ``spec``. The required and optional parameters
available depend on the ``kind`` of resource you are describing.

The pod spec we defined looked like this:

.. code-block:: yaml

    spec:
      containers:
        - name: hello
          image: ubuntu:22.04
          command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

There is just one stanza, the ``containers`` stanza, which is a list of containers (recall that pods can contain
multiple containers). Here we are defining just one container. For each container, we provide:

  * ``name`` (optional) -- this is the name of the container, similar to the name attribute in Docker.
  * ``image`` (required) -- the image we want to use for the container, just like with Docker.
  * ``command`` (optional) -- the command we want to run in the container. Here we are running a short BASH script.


Creating the Pod In K8s
-----------------------

We are now ready to create our pod in k8s. To do so, we use the ``kubectl apply`` command. In general, when you have
a description of a resource that you want to create or update in k8s, the ``kubectl apply`` command can be used.

In this case, our description is contained in a file, so we use the ``-f`` flag. Try this now:

.. code-block:: console

  [user-vm]$ kubectl apply -f pod-basic.yml

If all went well and k8s accepted your request, you should see an output like this:

.. code-block:: console

  pod/hello created


.. note:: 

  The message ``pod/hello created`` indicates that the description of the pod was valid, that
  k8s has saved the pod definition in its database and that it is working on starting the pod on the
  cluster. It does **not** mean the pod is already created/running on the cluster. 

In practice, we won't be creating many ``Pod`` resources directly -- we'll be creating other resources, such as
``deployments`` that are made up of pods -- but it is important to understand pods and to be able to work
with pods using ``kubectl`` for debugging and other management tasks.


.. note::

  The pod we just created is running on the k8s cluster, NOT on your student VM and NOT on kube controller
  node. You will not be able to find it using commands like docker ps, etc.

During the lecture, we'll go to the diagram to help explain what is going on.



Working With Pods
-----------------

We can use additional ``kubectl`` commands to get information about the pods we run on k8s.

Listing Pods
~~~~~~~~~~~~

For example, we can list the pods on the cluster with ``kubectl get <object_type>`` -- in this case, the object type
is "pods":

.. code-block:: console

   [user-vm]$ kubectl get pods
   NAME    READY   STATUS    RESTARTS   AGE
   hello   1/1     Running   0          99s

The output is fairly self-explanatory. We see a line for every pod which includes its name, status, the number of times
it has been restarted and its age. Our ``hello`` pod is listed above, with an age of ``99s`` because we just started it
but it is already ``Running``. 


Getting and Describing Pods
~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can pass the pod name to the ``get`` command -- i.e., ``kubectl get pods <pod_name>`` -- to just get information on
a single pod

.. code-block:: console

   [user-vm]$ kubectl get pods hello
   NAME    READY   STATUS    RESTARTS   AGE
   hello   1/1     Running   0          3m1s

The ``-o wide`` flag can be used to get more information:

.. code-block:: console

   [user-vm]$ kubectl get pods hello -o wide
   NAME    READY   STATUS    RESTARTS   AGE     IP              NODE                  NOMINATED NODE   READINESS GATES
   hello   1/1     Running   0          4m56s   10.233.97.195   coe332-kubernetes-4   <none>           <none>

Finally, the ``kubectl describe <resource_type> <resource_name>`` command gives additional information, including the
k8s events at the bottom. While we won't go into the details now, this information can be helpful when troubleshooting
a pod that has failed:

.. code-block:: console

   [user-vm]$ kubectl describe pods hello 
   Name:             hello
   Namespace:        USERNAME
   Priority:         0
   Service Account:  default
   Node:             coe332-kubernetes-4/129.114.36.49
   Start Time:       Tue, 02 Apr 2024 09:10:03 -0500
   Labels:           <none>
   Annotations:      cni.projectcalico.org/containerID: bea7a74509b2535625185701985ce4f3440a6f0e022ec9ba6a5cdee9b0886e06
                     cni.projectcalico.org/podIP: 10.233.97.195/32
                     cni.projectcalico.org/podIPs: 10.233.97.195/32
   Status:           Running
   IP:               10.233.97.195
   IPs:
     IP:  10.233.97.195
   Containers:
     hello:
       Container ID:  containerd://573593ae075052990c5bf12c8f0dd52c5a11ad1f1069a0b6ead84a080a08c443
       Image:         ubuntu:22.04
       Image ID:      docker.io/library/ubuntu@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e
       Port:          <none>
       Host Port:     <none>
       Command:
         sh
         -c
         echo "Hello, Kubernetes!" && sleep 3600
       State:          Running
         Started:      Tue, 02 Apr 2024 09:10:07 -0500
       Ready:          True
       Restart Count:  0
       Environment:    <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jkcd2 (ro)
   Conditions:
     Type              Status
     Initialized       True 
     Ready             True 
     ContainersReady   True 
     PodScheduled      True 
   Volumes:
     kube-api-access-jkcd2:
       Type:                    Projected (a volume that contains injected data from multiple sources)
       TokenExpirationSeconds:  3607
       ConfigMapName:           kube-root-ca.crt
       ConfigMapOptional:       <nil>
       DownwardAPI:             true
   QoS Class:                   BestEffort
   Node-Selectors:              <none>
   Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
   Events:                      <none>


Getting Pod Logs
~~~~~~~~~~~~~~~~

We can use ``kubectl logs <pod_name>`` command to get the logs associated with a pod:

.. code-block:: console

   [user-vm]$ kubectl logs hello
   Hello, Kubernetes!

Note that the ``logs`` command does not include the resource name ("pods") because it only can be applied to pods. The
``logs`` command in k8s is equivalent to that in Docker; it returns the standard output (stdout) of the container.


Using Labels
~~~~~~~~~~~~

In the pod above we used the ``metadata`` stanza to give our pod a name. We can use ``labels`` to add additional metadata
to a pod. A label in k8s is nothing more than a ``name: value`` pair that we create to organize objects in a 
meaningful way. We can choose any value for ``name`` and ``value`` that we wish but they must be strings. If you
want to use a number like "10" for a label name or value, be sure to enclose it in quotes (i.e., ``"10"``).

You can think of these ``name:value``
pairs as variables and values. So for example, you might create a label called ``shape`` with values 
``circle``, ``triangle``, ``square``, etc. A more realistic label might be ``component_type`` with 
values  ``api``, ``database``, ``worker``, etc. Or, you could imagine labeling pods as either
``production`` or ``development`` instances.
Multiple pods can have the same ``name:value`` label.

Let's use the pod definition above to create a new pod with a label.

Create a file called ``pod-labeled.yml``, open it up in an editor and paste the following code in:

.. code-block:: yaml
   :linenos:

   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: hello-labeled
     labels:
       version: "1.0"
   spec:
     containers:
       - name: hello
         image: ubuntu:22.04
         command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']

Let's create this pod using ``kubectl apply``:

.. code-block:: console

  [user-vm]$ kubectl apply -f pod-labeled.yml
  pod/hello-labeled created

Now when we list our pods, we should see it

.. code-block::

   [user-vm]$ kubectl get pods
   NAME            READY   STATUS    RESTARTS   AGE
   hello           1/1     Running   0          22m
   hello-labeled   1/1     Running   0          22s


Filtering By Labels With Selectors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Labels are useful because we can use ``selectors`` to filter our results for a given label name and value. To specify
a label name and value, use the following syntax: ``--selector "<label_name>=<label_value>"``.

For instance, we can search for pods with the version 1.0 label like so:

.. code-block:: console

  [user-vm]$ kubectl get pods  --selector "version=1.0"
   NAME          READY   STATUS    RESTARTS   AGE
   hello-label   1/1     Running   0          4m58s

We can also just use the label name to filter with the syntax ``--selector "<label_name>"``. This will find any pods with
the label ``<label_name>``, regardless of the value.


Exec into a Container inside a Pod
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Just like with the Docker CLI, the kubectl CLI provides a means to attach your terminal to a shell inside a 
running container. This is a useful way to debug a pod or test connections between pods. Assuming you have a
pod named `hello`, try:

.. code-block:: console

   [user-vm]$ kubectl exec -it hello -- /bin/bash
   root@hello:/#

From there, type ``logout`` or press ``Ctrl+D`` to return to the command line.



Delete a Pod
~~~~~~~~~~~~

Finally, we can delete a running pod using ``kubectl delete pods <pod_name>``:

.. code-block:: console

   [user-vm]$ kubectl delete pods hello
   pod "hello" deleted

The pod (and the container running inside the pod) has been removed from the k8s cluster. Another safe way to delete a pod
is to pass ``kubectl delete`` the YAML file describing the pod. In this case, you do not in include the resource name
("pods") because that is inferred from the contents of the YAML file.


.. code-block:: console

   [user-vm]$ kubectl delete -f pod-labeled.yml 
   pod "hello-labeled" deleted

We should now be able to see that our pods have all been deleted. It is important to periodically clean up and remove 
old resources to save ourselves resources (e.g. AWS credits) or in this case, it is good to clean up the k8s cluster
because it is a shared resource among the whole class.

.. code-block:: console

   [user-vm]$ kubectl get pods
   No resources found in USERNAME namespace.






EXERCISE
~~~~~~~~

Launch a new pod running Python v3.10.

Launch another pod running the API container from your latest homework.


Additional Resources
~~~~~~~~~~~~~~~~~~~~

 * `k8s Pod Reference <https://kubernetes.io/docs/concepts/workloads/pods/>`_
