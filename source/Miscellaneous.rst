Miscellaneous topics
====================


Migrating an instance to another project/cloud
----------------------------------------------

The migration of an instance from one project to another one, or to a
different OpenStack cloud, can be done using snapshots.

In short the procedure to migrate an instance is the following:

-  Create a snapshot of the instance in the source project, if possible 
   (please refer
   to the instructions provided at :ref:`Snapshotting Virtual Machines<SnapshottingVMs>` which also explain what are the restrictions of this procedure).

-  Transfer the snapshot from the source project to the destination one
   (this was discussed in the previous section:  :ref:`Migrating an image to another cloud<MigratingAnImageToAnotherCloud>`).

-  In the target environment boot a new instance from the snapshot.


