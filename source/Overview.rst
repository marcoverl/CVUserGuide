Overview of CloudVeneto
==========================

.. _tt:

**CloudVeneto** is an OpenStack-based cloud.

It allows the
instantiation of Virtual Machines (VMs) of the desired environments (in
terms of operating system, installed software, etc.) and of the desired
flavors (in terms of processors, memory size, etc.).
It also provides storage: block storage (volumes that can be attached to such virtual 
instances) and object storage.

CloudVeneto also offers higher level services, such as the orchestration of
multiple resources, the
provision of Kubernetes clusters, etc.

Even if it is a single, logical Cloud service, its resources are spread
in two different data centers: Padova (INFN Padova - University of Padova's
"Dipartimento di Fisica e Astronomia"), and INFN
Laboratori Nazionali di Legnaro (LNL).

The CloudVeneto is currently based on the *Xena* version of the OpenStack
middleware.


Who can use CloudVeneto ?
-------------------------
The CloudVeneto infrastructure is available to the users and collaborators
of the following Departments of the University of Padova:

* Dipartimento di Biologia (DiBio)
* Dipartimento di Fisica e Astronomia (DFA)
* Dipartimento di Geoscienze
* Dipartimento di Ingegneria Civile, Edile e Ambientale (DICEA)
* Dipartimento di Ingegneria dell'Informazione (DEI)
* Dipartimento di Matematica 
* Dipartimento di Medicina Molecolare (DMM)
* Dipartimento di Scienze Biomediche (DSB)
* Dipartimento di Scienze Chimiche (DiSC)
* Dipartimento di Scienze del Farmaco (DSF)

and INFN units:

* Sezione di Padova
* Laboratori Nazionali di Legnaro (LNL)


Resource users and administrators
---------------------------------
Please note that 
when you create a resource (e.g. a virtual machine) on CloudVeneto, you
are then responsible to manage it.
This means that you need to have adequate skills in using the resource
that you create and at least basic system administration
skills (some basics on Linux administration are presented in 
:ref:`Some basics on Linux administration<linuxbasics>`).

If you don't have such system administration skills, you may want to
ask a colleague to create and manage the needed resources.




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
.. _NetworkAccess:

Cloud instances are by default instantiated on private networks.



To access Cloud virtual machines it is therefore necessary to go 
through a gate machine.
A Cloud specific gate host (*gate.cloudveneto.it*) can be used.

The virtual instances created on INFN projects (and some Unipd DFA projects)
are by default “visible” from the Local Area Networks (LANs) of
both INFN Padova/Unipd Physics Dept. and INFN-LNL. This means that e.g. users can access via ssh
VMs of the Cloud directly from their desktops located in INFN 
Padova/Physics Dept. or INFN Legnaro without going through the gate machine.

If needed, e.g. if a VM should host a service accessible from the
Internet, such VM on the Cloud can be given a public IP. If this is
the case please contact support@cloudveneto.it.

It is possible to control which services/ports can be
accessed using the security groups (discussed :ref:`later<SecurityGroups>`) and firewalls on
the relevant VMs.

From a VM of the Cloud, it is possible to access the Internet. 


.. warning::
    By default from a VM of the Cloud it is not possible to access a host/service hosted in the INFN
    Padova or Legnaro LANs. If, for some reasons, you need to access some
    services hosted in INFN Padova or Legnaro from the Cloud, please contact
    support@cloudveneto.it.


Getting help
------------
In case of problems with the  CloudVeneto infrastructure, such as:

- Problems during the registration process

- Problems creating a virtual machine

- Problems creating a volume, or attaching it to an instance

you can contact the admins at support@cloudveneto.it.


Please provide all the information needed to debug the problem, e.g.:

- your cloud username
- the name of the project
- the IP number of the instance (in case of problems with a virtual machine)
- the image name and the flavor name (in case of problems creating an 
  instance)  


The CloudVeneto admins instead don't provide support to the virtual machines
instantiated on the Cloud: once you created a server on the Cloud, such server is
**managed by you**.


This implies that the CloudVeneto support crew **doesn't provide support** on
topics like:

-  how to install/compile/configure your software;

-  ssh / scp basic usage;

-  basic linux usage (some documentation is available on :ref:`Some basics on Linux administration<linuxbasics>`);

-  accessing your VM 'the graphical way'.


You might ask your Department / Institution technicians in case of problems with
your virtual machine that you are not able to solve on your own.



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


Getting help for INFN Padova users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`INFN-Padova computing and Network
service <https://www.pd.infn.it/eng/computing-and-networking/>`__ can provide
support to INFN-Padova users only for instances created
using the INFN-Padova "blessed" images, described in :ref:`Public Images for INFN Padova users<publicimagesPd>`.

When contacting the INFN-Padova computing and Network         
service to have support with a virtual machine,
please provide all the information needed to debug the problem, in
particular:

- The IP number of the instance
- The image name 
- The flavor name

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
   CloudVeneto.it", Published in: EPJ Web Conf. 214 (2019) 07010,
   DOI: 10.1051/epjconf/201921407010

-  Cloudveneto web site: http://cloudveneto.it


