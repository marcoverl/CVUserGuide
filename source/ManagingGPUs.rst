..    include:: <isonum.txt>

Managing GPUs
================
CloudVeneto provides Unipd Physics Dept. and INFN Padova users with some
GPUs (Graphics Processing Units). These are:

- 4 GPU Nvidia Tesla T4
- 1 GPU Nvidia Quadro RTX 6000 
- 2 GPU Nvidia TITAN Xp
- 1 GPU Nvidia GeForce GTX TITAN



Using a CloudVeneto GPU means accessing a virtual machine which has
full access and direct control of such GPU device.



Creating a GPU instance 
-----------------------
GPU instances, i.e. virtual machines which have access to one 
or more GPUs can be created only from the **HPC-Physics** project.

So, first of all, you need to request the affiliation to such project
(see :ref:`Apply for other projects<ApplyForOtherProjects>` for
the relevant instructions).

The instructions to then create a GPU instance are the very same for
the creation of a 'standard' virtual machine (see
:ref:`Creating Virtual Machines<creatingvms>`). You will only have to
pay attention to use one of these special flavors:


- **cloudveneto.15cores90GB20GB1T4**

  Flavor for an instance with 1 GPU Nvidia T4,
  15 VCPUs, 90 GB of RAM, 20 GB of ephemeral
  disk space.

- **cloudveneto.15cores90GB500GB1T4**

  Flavor for an instance with 1 GPU Nvidia T4,
  15 VCPUs, 90 GB of RAM, 500 GB of ephemeral
  disk space.

- **cloudveneto.30cores180GB500GB2T4**

  Flavor for an instance with 2 GPUs Nvidia T4,
  30 VCPUs, 180 GB of RAM, 500 GB of ephemeral
  disk space.

- **cloudveneto.60cores360GB500GB4T4**

  Flavor for an instance with 4 GPUs Nvidia T4,
  60 VCPUs, 360 GB of RAM, 500 GB of ephemeral
  disk space.


- **cloudveneto.8cores40GB500GB1Quadro**

  Flavor for an instance with 1 GPU Nvidia Quadro RTX 6000,
  8 VCPUs, 40 GB of RAM, 500 GB of ephemeral
  disk space.

- **cloudveneto.8cores40GB500GB1TitanXP**

  Flavor for an instance with 1 GPU Nvidia Titan Xp,
  8 VCPUs, 40 GB of RAM, 500 GB of ephemeral
  disk space.
 
- **cloudveneto.16cores80GB500GB2TitanXP**

  Flavor for an instance with 2 GPUs Nvidia Titan Xp,
  16 VCPUs, 80 GB of RAM, 500 GB of ephemeral
  disk space.

- **cloudveneto.4cores20GB150GB1GeforceGtx**

  Flavor for an instance with 1 GPU Nvidia GeForce GTX TITAN,
  4 VCPUs, 20 GB of RAM, 150 GB of ephemeral
  disk space.

Images for GPU instances
^^^^^^^^^^^^^^^^^^^^^^

You are responsible to create the image to be used (see 
:ref:`User Provided Images <userprovidedimages>` and 
:ref:`Building Images <buildingimages>`). 

`These <https://developer.nvidia.com/cuda-downloads>`__ 
instructions explain how to install CUDA toolkit and
the relevant drivers.

.. NOTE::

   For better performance, 
   we suggest to create images:

     - in raw format
     - setting the properties *hw_disk_bus=scsi* and *hw_scsi_model=virtio-scsi*, i.e., using the command line tool:

       ::

	  # glance image-update --property hw_disk_bus=scsi <image-id>
	  # glance image-update --property hw_scsi_model=virtio-scsi <image-id>

.. Just for reference, we provide a CentOS7.x image (**GPU-CentOS7-INFNPadova-x86_64-<date>**) which has the same content of the *CentOS7x-INFNPadova-x86-64-<date>* public images, and in addition provides the CUDA toolkit and the needed drivers. This image was tested with Nvidia T4 GPUs



Monitoring
----------
Unfortunately it is not straightforward to see which GPUs are being
used and which ones are available using the CloudVeneto Openstack dashboard.

You can refer to `this page <https://cloud-areapd.pd.infn.it/GPUs>`__ for
such information (please note that this page is updated every 30 minutes).


.. NOTE::

    If you need a GPU instance but the needed GPU(s) is/are allocated by other
    users, please contact support@cloudveneto.it.



Policies
--------
Please consider the following policies when using GPU instances:


- Since there is a high request to use GPUs, please **delete** your
  instance as soon as you don't need it anymore. 
  This is because virtual machines, even if 
  idle or in shutdown state, allocate resources (GPUs in particular) which 
  therefore aren't available to other users.

- Once activated, your virtual instance is **managed by you**. 



