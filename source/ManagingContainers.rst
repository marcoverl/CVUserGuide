..    include:: <isonum.txt>



Managing Containers
===================

A container is a standard unit of software that packages up everything 
needed to run an application: code, runtime, system tools, system libraries 
and settings. 
This allow applications to run reliably when moved from one computing environment to another.

Container engines, such as Docker, operate at the Operating System level, so they can be run inside a VM.

Running docker containers in the Cloud
--------------------------------------
For information about Docker and how to install it, please
look at the `Docker Documentation <https://docs.docker.com>`__.

We report here only about a couple of possible issues that need to be properly addressed.

The first one concerns networking. Docker by default sets the default MTU to 1500, but the default MTU for
Cloud virtual machines is reduced by the overhead added by some networking components.
Thus you need to reduce the Docker MTU to say 1420.

`Here <https://rahulait.wordpress.com/2016/02/28/modifying-default-mtu-for-docker-containers>`__
are the detailed steps on how to change the MTU for containers in docker.

The second issue concerns the file system: there could be problems running docker on old CentOS 7 releases 
where xfs is used as file system, as reported 
`here <https://www.pimwiddershoven.nl/entry/docker-on-centos-7-machine-with-xfs-filesystem-can-cause-trouble-when-d-type-is-not-supported>`__.
Newest CentOS 7 releases are not affected by this issue.

If, for some reasons, you can't use a newer version of CentOS 7 not affected
by this problem, the instructions reported 
`here <https://stackoverflow.com/questions/45461307/selinux-is-not-supported-with-the-overlay-graph-driver>`__ might help.



Orchestrating containers
------------------------
If you need to orchestrate multiple containers, we suggest to use Kubernetes (K8S), which is the most popular container orchestrator.
Chapter :ref:`Creating Kubernetes Clusters on the Cloud<CreatingK8s>` explains how to deploy a Kubernetes cluster on the Cloud.

