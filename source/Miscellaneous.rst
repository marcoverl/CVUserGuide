Miscellaneous topics
====================

Migrating an image to another cloud
-----------------------------------

Sometimes it becomes necessary to migrate an image (or snapshot) from
one OpenStack cloud to another one.

In short the procedure is the following:

**In the source cloud:**

-  Download the image/snapshot as a file

**In the destination cloud:**

-  Upload the downloaded image file to the new environment

To download the image from the source cloud, you need to use the
OpenStack CLI. After having sourced the environment script of the source
cloud (as explained in ?), run the **openstack image list** to find the
id of the image (photo-slave in the following example):

::

    $ openstack image list | grep photo-slave
    | 753e8fa1-3f1b-407d-9294-d22b89ec3184 | photo-slave                 | active |

Then use the **openstack image save** command, using the relevant id, to
download the image file:

::

    $ openstack image save --file photo-slave.qcow2 753e8fa1-3f1b-407d-9294-d22b89ec3184

To upload the image to the destination environment, first of all source
the environment script of the target cloud.

Then run the **openstack image create** command:

::

    $ openstack image create --private --disk-format=qcow2 \         
          --container-format=bare \                                                                                                               
          --file photo-slave.qcow2 photo-slave

Migrating an instance to another project/cloud
----------------------------------------------

The migration of an instance from one project to another one, or to a
different OpenStack cloud, can be done using snapshots.

In short the procedure to migrate an instance is the following:

-  Create a snapshot of the instance in the source project (please refer
   to the instructions provided at ?).

-  Transfer the snapshot from the source project to the destination one
   (this was discussed in the previous section: ?).

-  In the target environment boot a new instance from the snapshot.


