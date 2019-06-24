Overview of CloudVeneto
==========================

.. _tt:

**CloudVeneto** is an OpenStack based cloud.

It allows the
instantiation of Virtual Machines (VMs) of the desired environments (in
terms of operating system, installed software, etc.) and of the desired
flavors (in terms of processors, memory size, etc.).
It also provides storage volumes that can be attached to such virtual 
instances.

CloudVeneto also offers higher level services, such as the orchestration of
multiple resources, the instantiation of elastic batch systems, the
provision of Kubernetes clusters, etc.

Even if it is a single, logical Cloud service, its resources are spread
in two different locations: Padova (INFN Padova - University of Padova's
"Dipartimento di Fisica e Astronomia"), and INFN
Laboratori Nazionali di Legnaro (LNL).

The CloudVeneto is currently based on the *Rocky* version of the OpenStack
middleware.

Projects
--------
.. _projects:

**Projects** (also known as *tenants*) are organizational units in the
cloud. A project is used to map or isolate users and their resources.

Projects are given quotas on resource usage, in terms of virtual
machines, cores, memory, storage, etc.

A project can have a single user as member (*personal project*) but the
typical use case are *shared projects*, with multiple users as members,
which can map to experiments, organizations, research groups, etc. 

A
user can be on multiple projects at the same time and switch between
them.

In the CloudVeneto, projects usually map to experiments or other research
groups. Among the project's users there is a **project manager** (usually
the team leader) who is responsible to manage (accepting or refusing)
membership requests for the project.

.. warning::
    *Personal private* projects are discouraged and are created only
    for convincing reasons.

Network access
--------------

Cloud instances are by default “visible” from the Local Area Networks (LANs) of
both INFN Padova/Unipd Physics Dept. and INFN-LNL. This means that e.g. users can access via ssh
VMs of the Cloud directly from their desktops located in INFN 
Padova/Physics Dept. or
INFN Legnaro. It is then possible to control which services/ports can be
accessed using the security groups (discussed later) and firewalls on
the relevant VMs.

If it is necessary to log on a VM of the Cloud from a location different
than INFN Padova/Physics Dept. and INFN Legnaro, it is necessary to go 
through a gate machine.
A Cloud specific gate host (*gate.cloudveneto.it*) can be used.


If needed, e.g. if a VM should host a service accessible from the
Internet, such VM on the Cloud can be given a public IP. If this is
the case please contact support@cloudveneto.it.


From a VM of the Cloud, it is possible to access the Internet, while by
default it is not possible to access a host/service hosted in the INFN
Padova or Legnaro LANs. If, for some reasons, you need to access some
services hosted in INFN Padova or Legnaro from the Cloud, please contact
support@cloudveneto.it.


Getting help
------------

If you have problems using the CloudVeneto cloud, please contact the
admins at support@cloudveneto.it.

.. warning::

    The CloudVeneto provides and infrastructure where you can
    instantiate your servers but, once activated, your server are
    **managed by you**.

    This implies that the CloudVeneto support crew **doesn't provide support** on
    topics like:

    -  How to install/compile/configure your software;

    -  ssh / scp basic usage (ask your Department / Institution
       technicians about that);

    -  Basic linux usage (except what's described in this guide);

    -  Accessing your VM 'the graphical way'.

    `INFN-Padova computing and Network
    service <http://www.pd.infn.it/calcolo/indexEN.html>`__ can provide
    support only to INFN-Padova users and only for instances created
    using "blessed" images, (as described in :ref:`Public Images <publicimages>`).

Experiences, problems, best practices, etc. can be shared with the other
users of the CloudVeneto using the discuss@cloudveneto.it mailing list.
By default all CloudVeneto users are member of this mailing list. If you
want to be removed from this mailing list please send an e-mail to
majordomo@pd.infn.it, writing in the body of the mail:

::

    unsubscribe discuss <your-email-address>

Changes and planned interventions to the service will be posted on the
announce@cloudveneto.it. All registered users to the Cloud are member of
this mailing list.


Acknowledge CloudVeneto / Scientific citations
----------------------------------------------

We kindly ask you to acknowledge the usage of the CloudVeneto
infrastructure in any scientific publication or elsewhere. The following
quote can be used:

    *CloudVeneto is acknowledged for the use of computing and storage
    facilities.*

References:

-  P. Andreetto et
   al., "Merging OpenStack based private clouds: the case of 
   CloudVeneto.it", to appear in Proceedings of CHEP2018 conference

-  Cloudveneto web site: http://cloudveneto.it


