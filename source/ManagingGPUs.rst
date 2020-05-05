..    include:: <isonum.txt>

Managing GPUs
================
CloudVeneto provides Unipd Physics Dept. and INFN Padova users with some
GPUs (Graphics Processing Units). These are:

- 4 GPU Nvidia V100
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
The only exception is  is for the T4 GPUs that are usable also from the
**PhysicsOfData-students** project.

So, first of all, you need to request the affiliation to such project
(see :ref:`Apply for other projects<ApplyForOtherProjects>` for
the relevant instructions).

The instructions to then create a GPU instance are the very same for
the creation of a 'standard' virtual machine (see
:ref:`Creating Virtual Machines<creatingvms>`). You will only have to
pay attention to use one of these special flavors:


- **cloudveneto.18cores56GB20+75GB1V100**

  Flavor for an instance with 1 GPU Nvidia V100,
  18 VCPUs, 56 GB of RAM, 20 GB of ephemeral
  root disk space, 75 GB of extra ephemeral disk space.


- **cloudveneto.36cores112GB20+170GB2V100**

  Flavor for an instance with 2 GPU Nvidia V100,
  36 VCPUs, 112 GB of RAM, 20 GB of ephemeral root disk space,
  170 GB of extra ephemeral disk space.

- **cloudveneto.72cores224GB20+360GB4V100**

  Flavor for an instance with 4 GPU Nvidia V100,
  72 VCPUs, 224 GB of RAM, 20 GB of ephemeral root disk space,
  360 GB of extra ephemeral
  disk space.

- **cloudveneto.15cores90GB20+700GB1T4**

  Flavor for an instance with 1 GPU Nvidia T4,
  15 VCPUs, 90 GB of RAM, 20 GB of ephemeral root disk space,
  700 GB of extra ephemeral disk space.

- **cloudveneto.30cores180GB20+1400GB2T4**

  Flavor for an instance with 2 GPUs Nvidia T4,
  30 VCPUs, 180 GB of RAM, 20 GB of ephemeral root disk space,
  1400 GB of extra ephemeral disk space.

- **cloudveneto.60cores360GB20+2800GB4T4**

  Flavor for an instance with 4 GPUs Nvidia T4,
  60 VCPUs, 360 GB of RAM, 20 GB of ephemeral root
  disk space, 2800 GB of extra ephemeral disk space.

- **cloudveneto.8cores40GB20+500GB1Quadro**
  
  Flavor for an instance with 1 GPU Nvidia Quadro RTX 6000,
  8 VCPUs, 40 GB of RAM, 20 GB of ephemeral root
  disk space, 500 GB of extra ephemeral disk space.

- **cloudveneto.8cores40GB20+500GB1TitanXP**

  Flavor for an instance with 1 GPU Nvidia Titan Xp,
  8 VCPUs, 40 GB of RAM, 20 GB of ephemeral root
  disk space, 500 GB of extra ephemeral disk space.
 
- **cloudveneto.16cores80GB20+1000GB2TitanXP**

  Flavor for an instance with 2 GPUs Nvidia Titan Xp,
  16 VCPUs, 80 GB of RAM, 20 GB of ephemeral root
  disk space, 1000 GB of extra ephemeral disk space.

- **cloudveneto.4cores20GB20+200GB1GeforceGtx**

  Flavor for an instance with 1 GPU Nvidia GeForce GTX TITAN,
  4 VCPUs, 20 GB of RAM, 20 GB of ephemeral root
  disk space, 200 GB of extra ephemeral disk space.

.. WARNING ::
   When you snapshot an instance created using one of such flavors, please
   consider that only the root disk is saved. The content of the
   extra ephemeral disk is not saved !

.. WARNING ::

    For historical reasons there are still some instances created
    using flavors with a large root disk: since the size limit for images
    and snapshots is 25 GB, such
    instances cannot be snapshotted.
    

.. NOTE::

   Before allocation one or more GPUs, please register such allocation in
   `this document <https://tinyurl.com/yjz47dgx>`__. Please be sure to fill 
   also the 'estimated end date' field. 
   If the GPU(s) you want to use are not available, you may use that
   document to register a reservation. 
 

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



Just for reference, for the HPC-Physics project 
we provide a CentOS7.x image (**GPU-CentOS7-INFNPadova-x86_64-<date>**) which has the
same content of the *CentOS7x-INFNPadova-x86-64-<date>* public images, and in addition provides the CUDA 
toolkit and the needed drivers. This image was tested with Nvidia T4 GPUs
and with Nvidia V100 GPUs.

.. WARNING ::

    On a VM instantiated using this image, cuda is installed at the first boot (and its installation can 
    take several minutes).
    You may understand if the installation has been done if the following command:

       ::

          # rpm -q cuda-drivers

    returns something like.

       ::

          cuda-drivers-440.33.01-1.x86_64

    A further reboot might then be needed.


Monitoring
----------
Unfortunately it is not straightforward to see which GPUs are being
used and which ones are available using the CloudVeneto Openstack dashboard.

You can refer to `this page <https://cloud-areapd.pd.infn.it/GPUs>`__ for
such information (please note that this page is updated every 30 minutes).





Policies
--------
Please consider the following policies when using GPU instances:


- Since there is a high request to use GPUs, please **delete** your
  instance as soon as you don't need it anymore. 
  This is because virtual machines, even if 
  idle or in shutdown state, allocate resources (GPUs in particular) which 
  therefore aren't available to other users.

- Once activated, your virtual instance is **managed by you**. 

- Before allocation one or more GPUs, please register such allocation in
  `this document <https://tinyurl.com/yjz47dgx>`__. Please be sure to fill 
  also the 'estimated end date' field. Instances for which there isn't an 
  entry in this document can be deleted by the Cloud administrators.


- Please don't reserve the GPU(s) for long (i.e > 1 week) periods.

