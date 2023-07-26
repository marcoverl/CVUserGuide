Who can access my data stored on CloudVeneto ?
==============================================

Who can read the data stored on my virtual machine ?
----------------------------------------------------
.. _WhoCanReadOnMyVM:

When you create a VM (and therefore you become the owner of that VM)
using a CloudVeneto :ref:`Public image<publicimages>`,
by default you are the only one who can access to this VM.
The only exception to this rule applies to:

- the **<OperatingSystem>-INFNPadova-x86-64-<date>** images, as explained :ref:`below<infnpdimages_privacy>`.
- the **k8s-node** image, which grants access to the CloudVeneto administrators for debugging purposes


:ref:`Private<userprovidedimages>`, :ref:`shared<sharedimages>` and
:ref:`community<communityimages>` images could instead grant access by default also to 
other users:
it is then up to you to check if these images are appropriated for
your privacy requirements.


As owner of a VM you can then grant access to this VM to other people,
as explained in :ref:`Creating accounts on your Virtual Machine<CreatingAccounts>`.
Is this therefore your responsability to decide:

- who can access your VM
- which (and how) data can be accessed by the users who have access to the
  virtual machine
- if and how (e.g. via NFS) data stored on the VM can be accessed by users 
  of other virtual machines

This applies to data stored both on the ephemeral storage of the VM and to 
volume(s) attached to the virtual machine.



Who can read the data stored on a volume ?
-------------------------------------------

As explained in :ref:`Using (attaching) a Volume<CreateFS>`:

- only the owner of a volume can attach the volume to a VM

- a volume can be attached also to a VM belonging to a user different than
  the owner of the volume

Once a volume is attached to a VM, see :ref:`above<WhoCanReadOnMyVM>` wrt who can access the data
stored on this volume.

Who can read the data stored on a object storage bucket ?
--------------------------------------------------------- 
As explained in :ref:`Object Storage<ObjectStorage>`, if you create a bucket and upload some files on this 
container, also the other members of the project can see (and also delete) 
these files.

If this is not suitable for your use cases,
It is possible to have a personal object storage bucket.



Can the CloudVeneto administrators read my data ?
-------------------------------------------------

CloudVeneto administrators have
advanced permissions that technically allow to 
access to user contents and activity records.

Such permissions are however used only for the administration and operations
of the Cloud infrastructure, and for debugging purposes.


Who can read my data if I use a INFN-Padova public image ?
-------------------------
.. _infnpdimages_privacy:



As explained in :ref:`Public Images for INFN Padova users<PublicImagesPd>`, 
the **<OperatingSystem>-INFNPadova-x86-64-<date>** images allow INFN-Padova 
system administrators (see INFN-Padova computing and Network service) to log 
with admin privileges on the instances created using these images.
