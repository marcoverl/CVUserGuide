..    include:: <isonum.txt>


Managing Images
===============

In a cloud environment, Virtual Machines are instantiated from images.
These images are registered in an Image Management Service, in our case
provided by the **Glance** OpenStack component.


Public Images
-------------
.. _publicimages:


Some images in the CloudVeneto are provided by the Cloud administrators.
These images are public, and visible to all users. They appear with
**Visibility** equal to **Public** in the **Compute** |rarr| **Images** menu.



.. image:: ./images/public_images.png
   :align: center



Public Images for INFN Padova users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The **SL6x-INFNPadova-x86-64-<date>** and
**CentOS7x-INFNPadova-x86-64-<date>** images are basic SL6.x / CentOS
7.x images which also include *cloud-init* to perform contextualization
based on the user data specified when the VM are instantiated. They also
configure CVMFS and the relevant squid servers.

Such images also configure the Padova LDAP server for user
authentication. This means that it is just necessary to “enable” the
relevant accounts on the VM adding in the /etc/passwd file:

::

                                                                                         
    +name1::::::                                                                                 
    +name2::::::                                                                                 
    ...                                                                                          

and creating their home directories.

Changes done in /etc/passwd could not be applied immediately by the
system. In this case a:

::

                                                                                         
    nscd -i passwd                                                                               

should help.

You can access instances created using these images, through ssh using
the Cloud keypair, considering the 'root' account, e.g.:

::

    ssh -i ~/private/my_key root@10.64.17.3


.. NOTE ::
    The **SL6x-INFNPadova-x86-64-<date>** and
    **CentOS7x-INFNPadova-x86-64-<date>** images also allow INFN-Padova
    system administrators to log (with admin privileges) on the
    instance.

    `INFN-Padova computing and Network
    service <https://www.pd.infn.it/eng/computing-and-networking/>`__
    (supporto@pd.infn.it) can provide support only for instances created
    using such images (only to INFN-Padova users).

User Provided Images
--------------------
.. _userprovidedimages:

Users can provide their own images and upload them to the Cloud Image
service: these images are private, meaning that they are only available
to the users of the project they are uploaded for.

.. NOTE ::
    Users are not allowed to publish public (i.e. available to all
    projects) images.

Many open source projects such as Ubuntu and Fedora produce
pre-built images which can be used for certain clouds. If these are
available, it is much easier to use them compared to building your own.

-  `Fedora repository <https://getfedora.org/cloud/>`__

-  `Ubuntu repository <https://cloud-images.ubuntu.com/>`__

-  `CentOS 7 images <http://cloud.centos.org/centos/7/images/>`__



Using an Ubuntu image as an example, after you downloaded the image from
the relevant web site, to upload such image using the command line tools
(see :ref:`Accessing the Cloud with command line tools<accessingthecloudthroughcli>`)
you need to:

-  Authenticate to OpenStack using the openrc script:

   ::

       $ . demo-openrc.sh
             

-  Issue the following command:

   ::

       $ openstack image create --private --disk-format=qcow2 \
             --container-format=bare \
             --file bionic-server-cloudimg-amd64.img ubuntu-bionic
             

Once loaded, the image can then be used to create virtual machines.

.. NOTE ::
    Images are usually provided in several formats. The most used ones in Cloud
    environments are 'qcow2' and 'raw'. 

    When considering the format to choose, you should take into account how
    the instantiation of a virtual machine on a Cloud compute node works.
    When a virtual machine is instantiated, first
    of all the relevant image is staged to the target hypervisor, if needed 
    (i.e. if this is the first instance created on that compute node using 
    this image).
    Then the image is converted to raw, if it is not already in that
    format.

    A qcow2 image is usually much smaller with respect to the same image 
    provided in raw format.
    This means that:

    - the registration to the OpenStack image service for images in qcow2 
      format is faster with respect to images in raw format
    - the staging of images in qcow2 format to the target compute node
      is faster with respect to images in raw format

    However qcow2 images, as reported above, need to be translated into
    raw format, and this can take a non negligible time, in particular for
    images with a large virtual disk size. 



Some system software is delivered in ISO image format. For example,
these steps show how to create an image from the Freedos ISO available
from `here <http://www.freedos.org/download/download/fd11src.iso>`__

::

    $ openstack image create --private --disk-format=iso \
    --container-format=bare --file=fd11src.iso freedos11

.. NOTE ::
    In CloudVeneto image size is limited to 25 GB.


Sharing Images
--------------

As mentioned before, users are not allowed to publish public images.
However images can be shared between different projects. This is
currently only possible via the command line tools.

If an image has been uploaded to your currently active project, using
the procedure described in :ref:`User Provided Images <userprovidedimages>`, you can then use the
**openstack image add project** operation to share that image with
another project.

To share an image, first source the project profile for the project
containing the image you want to share (e.g. *Fedora-Cloud-Base-27*) and
find its id with the **openstack image list** command
(*d4b02b71-755e-47ad-bb27-1ea5c23bf7cb* in the example):

::

    $ openstack image list
    +--------------------------------------+----------------------+--------+
    | ID                                   | Name                 | Status |
    +--------------------------------------+----------------------+--------+
    | 7ebe160d-5498-477b-aa2e-94a6d962a075 | CentOS7              | active |
    | d4b02b71-755e-47ad-bb27-1ea5c23bf7cb | Fedora-Cloud-Base-27 | active |
    | 071668fc-dfeb-4956-996a-d2a487755709 | cirros               | active |
    +--------------------------------------+----------------------+--------+

You then need to change (to 'Shared') the visibility of the image:

::

    openstack image set --property visibility=shared d4b02b71-755e-47ad-bb27-1ea5c23bf7cb

You now need to find the id of the project you wish to share the image
with. This will generally be done by looking at the openrc file of that
project and finding the *OS_PROJECT_ID* variable (in this example, it
is e81df4c0b493439abb8b85bfd4cbe071).

To share the image with id d4b02b71-755e-47ad-bb27-1ea5c23bf7cb to the
project whose id is e81df4c0b493439abb8b85bfd4cbe071, use the command:

::

    $ openstack image add project d4b02b71-755e-47ad-bb27-1ea5c23bf7cb e81df4c0b493439abb8b85bfd4cbe071 
    +------------+--------------------------------------+
    | Field      | Value                                |
    +------------+--------------------------------------+
    | created_at | 2018-03-19T16:09:21Z                 |
    | image_id   | d4b02b71-755e-47ad-bb27-1ea5c23bf7cb |
    | member_id  | e81df4c0b493439abb8b85bfd4cbe071     |
    | schema     | /v2/schemas/member                   |
    | status     | pending                              |
    | updated_at | 2018-03-19T16:09:21Z                 |
    +------------+--------------------------------------+

.. NOTE ::
    Because of a bug in OpenStack this command could return an error message such as: 
    ::
      403 Forbidden: Not allowed to create members for image d4b02b71-755e-47ad-bb27-1ea5c23bf7cb. (HTTP 403)
    even if actually the command worked.


Then a member of the target project (with id e81df4c0b493439abb8b85bfd4cbe071
in our example) needs to accept the image:

::

    $ openstack image set --accept d4b02b71-755e-47ad-bb27-1ea5c23bf7cb

In the target project, the image will appear in the image list:


.. image:: ./images/SharedImage.png
   :align: center



Building Images
---------------

Users can also build custom images, that can then been uploaded in the
Cloud Image service as described in :ref:`User Provided Images <userprovidedimages>`.

There are several tools providing support for image creation. Some of
them are described in the `Openstack
documentation <http://docs.openstack.org/image-guide/content/ch_creating_images_automatically.html>`__.

One example is **virt-builder**, which is briefly described in the next
subsection. 


Please consider these guidelines when creating an image:

- Data inside images can be accessed by the other members of the project. 
  So please don't store sensitive information (e.g. password) in the image.
- Please make sure that the *cloud-init* and *cloud-utils* packages are installed. This is needed 
  for contextualisation (see  
  :ref:`Contextualisations<contextualisation>`), 
  which allows to customize the image after 
  installation.
- If you use Fedora, CentOS, or RHEL, the *cloud-utils-growpart* and
  *gdisk* packages must be installed, since they are needed for extending partitions. if you use Ubuntu or Debian, 
  the *cloud-initramfs-growroot* package, which supports resizing root partition on the first boot, must be installed.

.. NOTE ::
    Another possible way to create an image is:

    - instantiating a virtual machine using an already registered image
    - applying your own customization on this instance (e.g. installing new
      packages)
    - creating a snapshot of this image
    - using the snapshot to instantiate new virtual machines

    However this usually means creating very large and quite inefficient images.
    Therefore we **strongly** suggest instead to use the provided public images and applying your customizations
    using contextualization (see :ref:`Contextualisations<contextualisation>`) 
    or to create new images using one of the tools mentioned above. 
   
virt-builder
^^^^^^^^^^^^
Virt-builder is a command line tool
for quickly creating customized images.

It takes cleanly prepared, digitally signed OS templates and customizes them.

Documentation how to use virt-builder is provided in the relevant
`man pages <http://libguestfs.org/virt-builder.1.html/>`__.

Here we provide some examples.

To see the operating system available to install, please use the following 
command:

   ::

      virt-builder --list               


  The following command:
   
   ::

      virt-builder -v -x centos-7.7 \
        --uninstall "chrony" \
	--install "ntp,cloud-init,cloud-utils,cloud-utils-growpart,gdisk" \
        --timezone Europe/Rome" \
        --write '/etc/ntp.conf:server ntp.pd.infn.it' \
        --run-command 'systemctl enable ntpd' \
        --edit '/etc/resolv.conf:s/nameserver 10.0.2.3//' \
	--output c7.7-test-grow.qcow2 --format qcow2 --selinux-relabel


creates a centos 7.7 image, called *centos-7.7.qcow2* in qcow2 format, where 
the packages *ntp*, *cloud-init*, *cloud-utils*, *cloud-utils-growpart* and
*gdisk* are installed.

The above command removes from the image the package *chrony*.

The option *--timezone Europe/Rome* sets the timezone.

The string 'server ntp.pd.infn.it' is written into the file */etc/ntp.conf*.

The command 'systemctl enable ntpd' is also issued 
(to start the ntpd service at boot).

The option *--edit '/etc/resolv.conf:s/nameserver 10.0.2.3//'* is
used as a workaround to address a problem in the centos7 default
image (a wrong nameserver is defined in /etc/resolv.conf).


The following command will instead create an ubuntu 18.04 image, called 
*k8s-ubuntu-18.04.raw*, in raw format:

  ::

     virt-builder -v -x ubuntu-18.04 \
    --install "net-tools,vim,apt-transport-https,ca-certificates,curl,software-properties-common,docker.io,sudo,wget,rsync,cloud-init,cloud-utils" \
    --timezone Europe/Rome \
    --output k8s-ubuntu-18.04.raw --format raw \
    --run install.sh \
    --firstboot-command 'apt-get update' \
    --firstboot-command 'apt-get upgrade -y' \
    --selinux-relabel

Besides specifying a set of packages to be installed, the above command specifies that the script
*install.sh* must be run on the disk image.
The script can be used e.g. to install packages from an extra repository:

  ::
 
     $ cat install.sh 
     #!/bin/bash
     curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
     cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
     deb https://apt.kubernetes.io/ kubernetes-xenial main
     EOF
     apt-get update
     apt-get install -y kubelet kubeadm kubectl


The command also specifies that the commands *apt-get update* and *apt-get upgrade -y* must be executed at the first boot.



virt-builder is very easy to install (it is provided by the *libguestfs-tools-c* package)
and doesn't require root privileges.
If needed, access to an instance where virt-builder is installed can be provided:
in such case please contact support@cloudveneto.it.



Deleting Images
---------------

Images that are not used anymore can be deleted. Deletion of images is
permanent and cannot be reversed.

To delete an image, log in to the dashboard and select the appropriate
project from the drop down menu at the top left. On the **Compute** |rarr| **Images**
page select the images you want to delete and click **Delete Images**.
Confirm your action pressing the **Delete Images** again on the confirmation
dialog box.

.. WARNING ::
    Don't delete an image if there are virtual machines created
    using this image, otherwise these VMs won't be able to start if
    rebooted.
