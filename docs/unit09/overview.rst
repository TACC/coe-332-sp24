Unit 9: Container Orchestration
===============================

In this unit, we begin our study of container orchestration with the Kubernetes ("**k8s**") system. 
We have already built a small HTTP application in the REST architecture using the Flask framework.
This HTTP application makes use of a database to persist state. We have recently added a third
component - a worker container. This set up is very typical of a modern distributed system. As 
the number of components grows, the work required to deploy and maintain this system increases.
Container orchestration systems such as **k8s** aid us in this deployment and management effort
by allowing us to run our applications across a cluster of machines and use APIs to make changes
to the application deployment over time.

.. toctree::
   :maxdepth: 1

   orchestration
   k8s_intro
   pods
   deployments
   pvcs
   services
   k8s_cheat_sheet

..
   publicip
   

