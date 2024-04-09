Mounts, Volumes, and Persistent Volume Claims
=============================================

In this module, we discuss the Kubernetes abstraction called "persistent volume claims", or "pvc"s for short.
After working through this module, students should be able to:

* Describe a pvc in a yaml file and create the pvc on a Kubernetes cluster using ``kubectl``
* Exec into a pod that is part of a deployment
* Utilize Kubernetes mounts, volumes and persistent volume claims to persist application data across pod/container 
  restarts

Persistent Storage on the Kubernetes Cluster
--------------------------------------------

Some applications such as databases need access to storage where they can save data that will 
persist across container starts and stops. We saw how to solve this with Docker using a host bind mount.
With k8s, the pods (containers) get started automatically for us on different nodes in the clusters, 
so a mount from a host won't work. Which host would we use to store the files to be persisted?

The solution in k8s involves a combination of what are called volume mounts, volumes, and persistent 
volume claims. The basic idea is similar to that of a Docker host bind mount -- we'll be replacing 
some location in the container image with some data stored outside of the container. But in order to 
handle the fact that the application container could get started on different compute nodes, we'll 
utilize a backend "storage resource" which provides block storage over a network.  

Create a new file, ``deployment-pvc.yml``, with the following contents, replacing "<username>" 
with your username:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 23,26,28

   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-pvc-deployment
     labels:
       app: hello-pvc-app
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: hello-pvc-app
     template:
       metadata:
         labels:
           app: hello-pvc-app
       spec:
         containers:
           - name: hellos
             image: ubuntu:22.04
             command: ['sh', '-c', 'echo "Hello, Kubernetes!" >> /data/out.txt && sleep 3600']
             volumeMounts:
             - name: hello-<username>-data
               mountPath: "/data"
         volumes:
         - name: hello-<username>-data
           persistentVolumeClaim:
             claimName: hello-<username>-data

.. note:: 

   Be sure to replace **<username>** with your actual username in the YAML above. 

We have added a ``volumeMounts`` stanza to ``spec.containers`` and we added a ``volumes`` stanza to the ``spec``.
These have the following effects:

* The ``volumeMounts`` include a ``mountPath`` attribute whose value should be the path in the container that is to
  be provided by a volume instead of what might possibly be contained in the image at that path. Whatever is provided
  by the volume will overwrite anything in the image at that location.
* The ``volumes`` stanza states that a volume with a given name should be fulfilled with a specific persistentVolumeClaim.
  Since the volume name (``hello-<username>-data``) matches the name in the ``volumeMounts`` stanza, this volume will be
  used for the volumeMount.
* In k8s, a persistent volume claim makes a request for some storage from a storage resource configured by the k8s
  administrator in advance. While complex, this system supports a variety of storage systems without requiring the
  application engineer to know details about the storage implementation.

Note also that we have changed the command to redirect the output of the ``echo`` command to the file ``/data/out.txt``.
This means that we should not expect to see the output in the logs for pod but instead in the file inside the container.

However, if we create this new deployment and then list pods we see something curious:

.. code-block:: console

   [user-vm]$ kubectl apply -f deployment-pvc.yml
   [user-vm]$ kubectl get pods
   NAME                                    READY   STATUS    RESTARTS   AGE
   hello-deployment-6949f8ddbc-d6rqb       1/1     Running   0          3m13s
   hello-label                             1/1     Running   0          39m
   hello-pvc-deployment-7c5f879cd8-zpgq5   0/1     Pending   0          5s

Our "hello-deployment" pods are still running fine but our new "hello-pvc-deployment" pod is still in "Pending" status. It
appears to be stuck. What could be wrong?

We can ask k8s to describe that pod to get more details:

.. code-block:: console

   [user-vm]$ kubectl describe pods hello-pvc-deployment-7c5f879cd8-zpgq5
   Name:           hello-pvc-deployment-7c5f879cd8-zpgq5
   Namespace:      USERNAME
   Priority:       0
   Node:           <none>
   Labels:         app=hello-pvc-app
                   pod-template-hash=7c5f879cd8
   Annotations:    <none>
   <... some output omitted ...>
   Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s 
   Events:
   Type     Reason            Age   From               Message
   ----     ------            ----  ----               -------
   Warning  FailedScheduling  61s   default-scheduler  0/3 nodes are available: 3 persistentvolumeclaim "hello-USERNAME-data" not found. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.



At the bottom we see the "Events" section contains a clue: persistentvolumeclaim "hello-USERNAME-data" not found.

This is our problem. We told k8s to fill a volume with a persistent volume claim named "hello-USERNAME-data" but we
never created that persistent volume claim. Let's do that now!

Open up a file called ``pvc-basic.yml`` and copy the following contents, being sure to replace ``<username>``
with your TACC username:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 5

   ---
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: hello-<username>-data
   spec:
     accessModes:
       - ReadWriteOnce
     storageClassName: cinder-csi
     resources:
       requests:
         storage: 1Gi

.. note:: 

   Again, be sure to replace **<username>** with your actual username in the YAML above. 



We will use this file to create a persistent volume claim against the storage that has been set up in the TACC k8s
cluster. In order to use this storage, you do need to know the storage class (in this case, "cinder-csi", 
which is the storage class for utilizing the Cinder storage system), and how much you want to request (in this case, just 1 Gig), but you
don't need to know how the storage was implemented.

.. note::

   Different k8s clusters may offer persistent storage that utilize different storage classes. Within TACC, 
   we also have k8s clusters that utilize the ``rbd`` and ``nfs`` storage classes, for example. Be sure to check with the
   k8s administrators to see what storage class(es) might be available.

We create this pvc object with the usual ``kubectl apply`` command:

.. code-block:: console

   [user-vm]$ kubectl apply -f hello-pvc.yml
   persistentvolumeclaim/hello-USERNAME-data created

Great, with the pvc created, let's check back on our pods:

.. code-block:: console

  [user-vm]$ kubectl get pods
   NAME                                    READY   STATUS        RESTARTS   AGE
   hello-deployment-9794b4889-mk6qw        1/1     Running       46         46h
   hello-deployment-9794b4889-sx6jc        1/1     Running       46         46h
   hello-deployment-9794b4889-v2mb9        1/1     Running       46         46h
   hello-deployment-9794b4889-vp6mp        1/1     Running       46         46h
   hello-pvc-deployment-ff5759b64-sc7dk    1/1     Running       0          45s

Like magic, our "hello-pvc-deployment" now has a running pod without us making any additional API calls to k8s!
This is the power of the declarative aspect of k8s. When we created the hello-pvc-deployment, we told k8s to always
keep one pod with the properties specified running at all times, if possible, and k8s continues to try and implement our
wishes until we instruct it to do otherwise.

.. note::

   You cannot scale a pod with a volume filled by a persistent volume claim. 


Exec Commands in a Running Pod
------------------------------

Because the command running within the "hello-pvc-deployment" pod redirected the echo statement to a file, the
hello-pvc-deployment-ff5759b64-sc7dk will have no logs. (You can confirm this is the case for yourself using the ``logs``
command as an exercise).

In cases like these, it can be helpful to run additional commands in a running pod to explore what is going on.
In particular, it is often useful to run shell in the pod container.

In general, one can run a command in a pod using the following:

.. code-block:: console

   [user-vm]$ kubectl exec <options> <pod_name> -- <command>

To run a shell, we will use:

.. code-block:: console

   [user-vm]$ kubectl exec -it <pod_name> -- /bin/bash

The ``-it`` flags might look familiar from Docker -- they allow us to "attach" our standard input and output to the
command we run in the container. The command we want to run is ``/bin/bash`` for a shell.

Let's exec a shell in our "hello-pvc-deployment-ff5759b64-sc7dk" pod and look around:

.. code-block:: console

   [user-vm]$ kubectl exec -it  hello-pvc-deployment-5b7d9775cb-xspn7 -- /bin/bash
   root@hello-pvc-deployment-5b7d9775cb-xspn7:/#

Notice how the shell prompt changes after we issue the ``exec`` command -- we are now "inside" the container, and our
prompt has changed to "root@hello-pvc-deployment-5b7d9775cb-xspn" to indicate we are the root user within the container.

Let's issue some commands to look around:

.. code-block:: bash

   [container]$ pwd
   /
   # exec put us at the root of the container's file system

   [container]$ ls -l
   total 8
   drwxr-xr-x   2 root root 4096 Jan 18 21:03 bin
   drwxr-xr-x   2 root root    6 Apr 24  2018 boot
   drwxr-xr-x   3 root root 4096 Mar  4 01:06 data
   drwxr-xr-x   5 root root  360 Mar  4 01:12 dev
   drwxr-xr-x   1 root root   66 Mar  4 01:12 etc
   drwxr-xr-x   2 root root    6 Apr 24  2018 home
   drwxr-xr-x   8 root root   96 May 23  2017 lib
   drwxr-xr-x   2 root root   34 Jan 18 21:03 lib64
   drwxr-xr-x   2 root root    6 Jan 18 21:02 media
   drwxr-xr-x   2 root root    6 Jan 18 21:02 mnt
   drwxr-xr-x   2 root root    6 Jan 18 21:02 opt
   dr-xr-xr-x 887 root root    0 Mar  4 01:12 proc
   drwx------   2 root root   37 Jan 18 21:03 root
   drwxr-xr-x   1 root root   21 Mar  4 01:12 run
   drwxr-xr-x   1 root root   21 Jan 21 03:38 sbin
   drwxr-xr-x   2 root root    6 Jan 18 21:02 srv
   dr-xr-xr-x  13 root root    0 May  5  2020 sys
   drwxrwxrwt   2 root root    6 Jan 18 21:03 tmp
   drwxr-xr-x   1 root root   18 Jan 18 21:02 usr
   drwxr-xr-x   1 root root   17 Jan 18 21:03 var
   # as expected, a vanilla linux file system.
   # we see the /data directory we mounted from the volume...

   [container]$ ls -l data/out.txt
   -rw-r--r-- 1 root root 19 Mar  4 01:12 data/out.txt
   # and there is out.txt, as expected

   [container]$ cat data/out.txt
   Hello, Kubernetes!
   # and our hello message!

   [container]$ exit
   # we're ready to leave the pod container

.. note::

   To exit a pod from within a shell (i.e., ``/bin/bash``) type "exit" at the command prompt.

.. note::

   The ``exec`` command can only be used to execute commands in *running* pods.


Persistent Volumes Are... Persistent
------------------------------------

The point of persistent volumes is that they live beyond the length of one pod. Let's see this in action. Do the
following:

1. Delete the "hello-pvc" pod. What command do you use?
2. After the pod is deleted, list the pods again. What do you notice?
3. What contents do you expect to find in the ``/data/out.txt`` file? Confirm your suspicions.


*Solution*:

.. code-block:: bash

   [user-vm]$ kubectl delete pods hello-pvc-deployment-5b7d9775cb-xspn7
   pod "hello-pvc-deployment-5b7d9775cb-xspn7" deleted

   [user-vm]$ kubectl get pods
   NAME                                    READY   STATUS              RESTARTS   AGE
   hello-deployment-9794b4889-mk6qw        1/1     Running             47         47h
   hello-deployment-9794b4889-sx6jc        1/1     Running             47         47h
   hello-deployment-9794b4889-v2mb9        1/1     Running             47         47h
   hello-deployment-9794b4889-vp6mp        1/1     Running             47         47h
   hello-pvc-deployment-5b7d9775cb-7nfhv   0/1     ContainerCreating   0          46s
   # wild -- a new hello-pvc-deployment pod is getting created automatically!

   # let's exec into the new pod and check it out!
   [user-vm]$ kubectl exec -it hello-pvc-deployment-5b7d9775cb-7nfhv -- /bin/bash

   [container] $ cat /data/out.txt
   Hello, Kubernetes!
   Hello, Kubernetes!  


.. warning::

   Deleting a persistent volume claim deletes all data contained in all volumes filled by the PVC permanently! This cannot
   be undone and the data cannot be recovered!


Additional Resources
--------------------

 * `Persistent Volumes <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>`_
 * `Storage Classes in k8s <https://kubernetes.io/docs/concepts/storage/storage-classes>`_
