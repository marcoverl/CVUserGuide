..    include:: <isonum.txt>


Creating Batch Clusters on the Cloud
====================================

The virtual machines provided by CloudVeneto can also be used to
implement batch clusters where users can run their applications (normal
jobs or docker containers).

In this chapter we explain how to implement a dynamic batch cluster
based on `HTCondor <http://research.cs.wisc.edu/htcondor/manual/>`__ and
elastiq using ECM.

Intro: the idea
---------------

You create on the cloud a virtual machine that acts as a master for a
dynamic batch system (implemented using HTCondor). When you create the
master you will need to describe the cluster configuration, as described
in ?.

The master node will be able to spawn new slave nodes (where jobs are
executed) when jobs are submitted to the batch system. The elastic
cluster will provide a number of virtual resources that scales up or
down depending on your needs. The total number of active virtual nodes
is dynamic.

The master node will act also as submitting machine: you can log in on
this node and submit jobs to the batch system. These jobs will run on
the slave nodes, get done, and eventually the slaves will be released.

.. NOTE ::
    The master can use a different flavor with respect to the slave
    nodes.

Creating Batch Clusters on the Cloud with HTCondor
--------------------------------------------------

When you create the master, using the instructions reported in ?, you
will need to specify some user-data to describe the cluster
configuration, as described below.

.. NOTE ::
    The master and the slaves must use the same image (which should have
    all the needed software).

Prerequisites
^^^^^^^^^^^^^

-  You should be registered in the Cloud as member of a project.

   .. WARNING::
       The project where you want to create the cluster should have only
       one available network (i.e the lan) otherwise, due to a bug with
       ec2api, the instantiation of slaves fails.

-  You need to have created a SSH key-pair, ad explained in ?. This will
   allow you to log in the master and slave nodes.

-  You need to download the EC2 credentials of the project you want to
   use (see ?). You can download them from the dashboard as following:

   Open the dashboard, select the project (drop down menu on top left of
   the dashboard), go to **Compute** |rarr| **API Access** and here click on **Download
   EC2 credentials**.

   You'll get a zip file with a name like: *testing-x509.zip* (in this
   case *testing* is the name of the project). The zip contains the
   following files:

   ::

       $ unzip testing-x509.zip
       Archive:  testing-x509.zip
       extracting: pk.pem
       extracting: cert.pem
       extracting: cacert.pem
       extracting: ec2rc.sh
             

   Extract all these files somewhere safe. The content of your ec2rc.sh
   file is something like:

   ::

       $ cat ec2rc.sh

       #!/bin/bash

       NOVARC=$(readlink -f "${BASH_SOURCE:-${0}}" 2>/dev/null) || NOVARC=$(python -c 'import os,sys; print os.path.abspath(os.path.realpath(sys.argv[1]))' "${BASH_SOURCE:-${0}}")
       NOVA_KEY_DIR=${NOVARC%/*}
       export EC2_ACCESS_KEY=
       export EC2_SECRET_KEY=
       export EC2_URL=https://cloud.cedc.csia.unipd.it:8773/services/Cloud
       export EC2_USER_ID=42 # nova does not use user id, but bundling requires it
       export EC2_PRIVATE_KEY=${NOVA_KEY_DIR}/pk.pem
       export EC2_CERT=${NOVA_KEY_DIR}/cert.pem
       export NOVA_CERT=${NOVA_KEY_DIR}/cacert.pem
       export EUCALYPTUS_CERT=${NOVA_CERT} # euca-bundle-image seems to require this set

       alias ec2-bundle-image="ec2-bundle-image --cert ${EC2_CERT} --privatekey ${EC2_PRIVATE_KEY} --user 42 --ec2cert ${NOVA_CERT}"
       alias ec2-upload-bundle="ec2-upload-bundle -a ${EC2_ACCESS_KEY} -s ${EC2_SECRET_KEY} --url ${S3_URL} --ec2cert ${NOVA_CERT}"
             

   where EC2\_ACCESS\_KEY and EC2\_SECRET\_KEY are different for each
   project and user. The elastiq service uses these values to
   instantiate and to kill VMs in the specific project.

-  You need to identify the image to be used for the master and for the
   slaves. Currently supported operating systems are RHEL6.x and
   derivates (SL6.x, CentOS7.x, etc.), RHEL7.x and derivates and Ubuntu.
   uCernVM based images are also supported. For such image you also need
   to know the relevant EC2 (AMI) id (see ?).

-  You need to set a specific security group to be used for the master
   node. This security group must include the following rules:

   ::

       Direction    Ether Type    IP Protocol    Port Range         Remote IP Prefix
       Egress       IPv4          Any            Any                0.0.0.0/0  
       Egress       IPv6          Any            Any                ::/0   
       Ingress      IPv4          ICMP           Any                0.0.0.0/0  
       Ingress      IPv4          TCP            22 (SSH)           0.0.0.0/0  
       Ingress      IPv4          TCP            9618               0.0.0.0/0  
       Ingress      IPv4          TCP            41000 - 42000      0.0.0.0/0
             

   ..NOTE::
       Instead of modifying the rules of an existing security group, we
       suggest to create a new security group named e.g.
       "master\_security\_group". Security groups are discussed in ?.

   The slave nodes will instead use the *default* security group of
   your project. This group must include the following rule:

   ::

       Direction  Ether Type  IP Protocol  Port Range   Remote IP Prefix   Remote Security Group
       Ingress    IPv4        Any          Any          -                  <master_security_group>
               

   where *<master_security_group>* is the name of the security group
   that was chosen for the master node.

-  You need to download the
   `ECM <https://github.com/CloudPadovana/ECM.git>`__ software. As
   explained in ?, this will be used to create the batch cluster
   configuration:

   ::

       $ git clone https://github.com/CloudPadovana/ECM.git
               

-  You need to install the euca2ools package to discover the EC2 (AMI)
   id of images. Euca commands are also used by ECM to show the list of
   available images for the cluster.

   ::

       yum install euca2ools (on RHEL/CentOS)
       apt-get install euca2ools (on Ubuntu)
                   

The cluster configuration
^^^^^^^^^^^^^^^^^^^^^^^^^

You must properly configure the ecm.conf file stored in the ECM
directory (created when you downloaded via git the ECM software)

::

    $ cat ecm.conf                                                                  
    FLAVOR_VMS=                               
    MAX_VMS=                                      
    MIN_VMS=                                      
    JOBS_PER_VM=                              
    IDLE_TIME=                                  
    KEY_NAME==
          

Where:

-  **<FLAVOR_VMS>** is the name of the flavor to be used for the slave
   nodes. Flavors have been discussed in ?. Available flavors are listed
   in the dashboard when you try to launch a VM.

-  **<MAX_VMS>** is the maximum number of slave nodes that can be
   instantiated.

-  **<MIN_VMS>** is the minimum number of slave nodes (never terminated,
   always available).

-  **<JOBS_PER_VM>** is the maximum number of jobs that will be run in a
   single slave node.

   .. IMPORTANT ::
       You have to verify that the number of jobs per VM is compatible
       with the number of VCPUs of the selected flavor.

-  **<IDLE_TIME>** is the time (in seconds) after which inactive VMs
   will be killed.

-  **<KEY_NAME>** is the name (without the .pem extension) of ssh key
   previously created (see ?) to be injected in the batch cluster nodes.

.. NOTE ::
    The batch system will use each CPU as a separate job slot. So if you
    have a flavor with 4 VCPUs, and you submit 1 job, the master will
    create 1 slave and use 1 of the 4 available VCPUs. If you submit 4
    jobs, again the master will create 1 slave, and will use all the 4
    VCPUs. Large flavors means less machines to be created but possibly
    a sub-optimal usage of resources.

Start the elastic cluster
^^^^^^^^^^^^^^^^^^^^^^^^^

To start the elastic cluster, you only need to instantiate the master
node.

When you create such master, you will need to specify some user-data to
describe the cluster configuration. The ecm.py script will create such
user-data file for you, using as input the ecm.conf file you previously
edited. (see ?).

First of all you have to set the relevant EC2 credentials:

::

    $ source ec2rc.sh
          

Then you must launch ecm.py file and follow the instructions. We will
create a CentOS 6 cluster as an example.

::

    $ python ecm.py

    Choose the Operating System (OS) you want 
    to use for your master and worker nodes:

    1: Fedora
    2: Ubuntu
    3: uCernVM
    4: CentOS7
    5: CentOS6

    OS type => 5
          

.. IMPORTANT ::
    The same OS will be used to instantiate both master node and worker
    node(s).

::

    Select the image for your CentOS6 based master and your CentOS6 based WNs:
    1: CentOS 6
    2: Other image. [WARNING] You have to know the EC2-id of image

    Image => 1

          

.. WARNING ::
    If you choose "Other image" you have to manually insert the image id
    in EC2 format (see ?).

The script will now print something like:

::

          Now you can use the master-centos6-2017-03-16-17.45.31 file to instantiate the master node of your elastiq cluster.
        

Now you have to start the master node. As explained in ?. go to
'Instances' and create a new instance with [Launch Instances].

In the details select:

[details]

-  *Instance Name* => whatever you like

-  *Flavor* => whatever you like; can be different from the flavor chosen
   for the slave nodes

-  *Image name* => The same image chosen for the slaves.

[Access and Security]

-  *Key pair* => The key-pair that will be used to log on the nodes of the
   batch cluster

-  *Security Group* => the security group for the master (choose only this
   one)

[post creation]

-  *Customization Script Source* => Select "File" from the dropdown menu
   and use "Choose File" to upload the user\_data\_file created by the
   ecm.py script

Then press launch.

Once you requested the creation of the master node, after some minutes,
you will see that the master virtual machine and some (depending on the
"MIN\_VMS" attribute you defined in the ecm.conf) slave nodes are
created.

Get the IP address of master, and log in on this machine using the key
you have imported/created before i.e:

::

    $ ssh -i ~/.ssh/id_rsa root@10.64.YY.XXX
        

For security reasons, as root you can not submit jobs to the HTCondor
cluster. So make sure that a 'normal' account exists in the master node.
In case you can create it using the command:

::

    # adduser <username>
        

Create a password for this account:

::

    # passwd <username>
        

You have to import any external disk, create homes, etc, as you would do
in a normal machine.

How slave nodes are installed and configured
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The instantiation of slaves nodes is managed by the elastiq service
running in the master node.

The min and max number of nodes is set by user in the ecm.conf: the
total number of active nodes will change dynamically with jobs need.

The installation of condor and its configuration on slaves is
automatically provided by ECM. Inside the user\_data\_file created by
the ecm.py script there is the parameter SLAVE\_USERDATA whose value is
the script for the installation and configuration of the slave, coded in
b64. The original uncoded script used for condor installation and
configuration on slaves is stored in the ECM/slave\_files directory.
There is a file for each Operating system, depending on the system
selected for the master. These files provide the basic configuration for
condor, to support both the Vanilla and Docker universes.

Generally the user doesn't need to modify these files. If, for some
reasons, you need to modify the condor configuration or you need to
install additional packages on the slaves, this script can be modified:
ECM will take care to code it in b64 and add the new value to the
SLAVE\_USERDATA parameter in the user\_data\_file.

Use the elastic cluster
^^^^^^^^^^^^^^^^^^^^^^^

Log to the master node using your unpriviledged account. Check if condor
is running with the command:

::

    $ condor_q
    -- Schedd: 10-64-20-181.virtual-analysis-facility : <10.64.20.181:41742>
    ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD

    0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
        

Check the status of the cluster:

::

    $ condor_status
    Name               OpSys      Arch   State     Activity LoadAv Mem   ActvtyTime

    slot1@10-64-22-84. LINUX      X86_64 Unclaimed Idle      0.000 1977  2+12:44:58
    slot2@10-64-22-84. LINUX      X86_64 Unclaimed Idle      0.000 1977  2+12:45:25
    Total Owner Claimed Unclaimed Matched Preempting Backfill

    X86_64/LINUX     2     0       0         2       0          0        0

    Total            2     0       0         2       0          0        0
        

Create you HTCondor 'job file'. A simple example is the following:

::

    $ cat test.classad
        
    Universe = vanilla
    Executable = /home/<username>/test.sh
    Log = test.log.$(Cluster)$(Process)
    Output = test.out.$(Cluster)$(Process)
    Error = test.err.$(Cluster)$(Process)
    Queue <number_of_jobs_to_submit>
          

where *test.sh* is the executable you want to run.

Submit your jobs issuing the command:

::

    $ condor_submit test.classad
        

and check their status with:

::

    $ condor_q
        

You can find documentation about HTCondor
`here <http://research.cs.wisc.edu/htcondor/manual/>`__.

Use the elastic cluster to run docker containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The HTCondor elastic cluster can also be used to run docker containers.
You don't need to install docker on your images: this is done by ECM.

Once the cluster is created, check if Docker is enabled on the slaves:

::

                                                                                                     
    # condor_status -l | grep -i Docker                                                                      
    StarterAbilityList = "HasTDP,HasEncryptExecuteDirectory,HasFileTransferPluginMethods,HasJobDeferral,HasJICLocalConfig,HasJICLocalStdin,HasPerFileEncryption,HasDocker,HasFileTransfer,HasReconnect,HasVM,HasMPI,HasRemoteSyscalls,HasCheckpointing"                                                                      
    DockerVersion = "Docker version 1.7.1, build 786b29d/1.7.1"                                              
    HasDocker = true                                                                                         
    StarterAbilityList = "HasTDP,HasEncryptExecuteDirectory,HasFileTransferPluginMethods,HasJobDeferral,HasJICLocalConfig,HasJICLocalStdin,HasPerFileEncryption,HasDocker,HasFileTransfer,HasReconnect,HasVM,HasMPI,HasRemoteSyscalls,HasCheckpointing"                                                                      
    DockerVersion = "Docker version 1.7.1, build 786b29d/1.7.1"                                              
    HasDocker = true                                                                                         
        

The following is a simple example which runs a docker container, which
is downloaded from docker-hub:

::

        
    $ cat test-docker.classad                                                                                
              
    universe = docker                                                                                        
    docker_image = debian                                                                                    
    executable = /bin/cat                                                                                    
    arguments = /etc/hosts                                                                                   
    should_transfer_files = YES                                                                              
    when_to_transfer_output = ON_EXIT                                                                        
    Log = test-docker.log.$(Cluster).$(Process)                                                              
    Output = test-docker.out.$(Cluster).$(Process)                                                           
    Error = test-docker.err.$(Cluster).$(Process)                                                            
    request_memory = 100M                                                                                    
    Queue <number_of_jobs_to_submit>
        

How to find the EC2 (AMI) id of an image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As explained above, to use the Elastic batch cluster you need to know
the EC2 (AMI) id of the image you want to use.

First of all you need to install the euca2ools and to download the EC2
credentials for your project, as explained in ?.

Uncompress (unzip) the EC2 zip file, and source the ec2rc.sh script to
set the correct environment:

::

    $ source ec2rc.sh
        

To discover the EC2 id of your image or snapshot, use the
'euca-describe-images' command:

::

    $ euca-describe-images -I ${EC2_ACCESS_KEY} -S ${EC2_SECRET_KEY} -U ${EC2_URL}

    IMAGE   ami-0000031b    None (uCernVM 2.3-0)    beaeede3841b47efb6b665a1a667e5b1        available       public                  machine                         instance-store
    IMAGE   ami-00000447    snapshot        36b1ddb5dab8404dbe7fc359ec95ecf5        available       public                  machine                         instance-store
        

In the example above:

-  the EC2 image id of the uCernVM 2.3-0 image is ami-0000031b

-  there is a snapshot whose EC2 is is ami-00000447.

   .. NOTE::
    In case you have snapshot on the output of the euca-describe-images
    you notice that you have no name associated with the ami-id. To
    obtain a nicely formatted list of (ami-id, name) couples you can use
    the following command:

    ::

        $ euca-describe-images --debug 2>&1 | grep 'imageId\|name' | sed 'N;s/\n/ /'

        <imageId>ami-00000002</imageId>       <name>cirros</name>
        <imageId>ami-0000000d</imageId>       <name>Fedora 20 x86_64</name>
        <imageId>ami-00000010</imageId>       <name>Centos 6 x86_64</name>
        <imageId>ami-00000013</imageId>       <name>Centos 7 x86_64</name>
        <imageId>ami-0000001b</imageId>       <name>ubuntu-14.04.3-LTSx86_64</name>
        <imageId>ami-0000005d</imageId>       <name>matlab-2015a-glnxa64</name>
        <imageId>ami-00000057</imageId>       <name>Win7-Pro-X86_64-ENU</name>
        <imageId>ami-00000027</imageId>       <name>Fedora 23 x86_64</name>
        <imageId>ami-00000069</imageId>       <name>Win7-Photoscan</name>
        <imageId>ami-0000002d</imageId>       <name>photo-slave</name>
        <imageId>ami-0000004e</imageId>       <name>s-medium-snap</name>
        <imageId>ami-0000005a</imageId>       <name>uCernVM 3.6.5</name>
        <imageId>ami-000000ae</imageId>       <name>archlinux</name>
        <imageId>ami-000000c6</imageId>       <name>ubuntu-16.04.1-LTS x86_64</name>
        <imageId>ami-000000c9</imageId>       <name>Fedora 25 x86_64</name>
        <imageId>ami-000000d2</imageId>       <name>x2go-thinclient-server</name>
        <imageId>ami-000000d5</imageId>       <name>Win7-test</name>
        <imageId>ami-000000db</imageId>       <name>matlab-2016b</name>
        <imageId>ami-000000d8</imageId>       <name>ubuntu-16.04.1+Matlab_2016b</name>
        ...
        Note that items may appear multiple times...
        ...
            

You can also see all the information of an image, e.g.:

::

    $ euca-describe-images --debug ami-00000447 
        

or:

::

    $ euca-describe-images -I ${EC2_ACCESS_KEY} -S ${EC2_SECRET_KEY} -U ${EC2_URL} --debug ami-00000447 
        

The returned output will be something like:

::

    <requestId>req-c56c3694-c555-464a-b21d-2c86ccc020be</requestId>
    <imagesSet>
    <item>
    <description/>
    <imageOwnerId>36b1ddb5dab8404dbe7fc359ec95ecf5</imageOwnerId>
    <isPublic>true</isPublic>
    <imageId>ami-00000447</imageId>
    <imageState>available</imageState>
    <architecture/>
    <imageLocation>snapshot/imageLocation>
    <rootDeviceType>instance-store</rootDeviceType>
    <rootDeviceName>/dev/sda1</rootDeviceName>
    <imageType>machine</imageType>
    <name>cernvm_230_ldap_pd</name>
    </item>
    </imagesSet>
    </DescribeImagesResponse>
        

Restart the elastic cluster
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes the steps to be followed to restart the elastic
cluster (this might be needed e.g. after a reboot).

-  If possible, delete all the slave nodes via
   `dashboard <https://cloud-areapd.pd.infn.it/dashboard>`__;

-  Log on the master node as root;

-  Check if both condor and elastiq services are already running

   ::

       # service condor status
       # service elastiq status
               

   In this case new slaves will be created and will join the cluster in
   some minutes.

-  if only condor service is running but elastiq isn't, please restart
   elastiq using the following command:

   ::

       # service elastiq start
               

   or

   ::

       # elastiqctl restart
               

   and wait for the creation of new slaves that will connect to the
   cluster in some minutes;

-  If condor isn't running and some elastiq processes are up and
   running, kill them:

   ::

       # ps -ef | grep elastiq
       # kill -9 <n_proc>
               

   and start the condor service using the following command:

   ::

       # service condor start
               

   Now the condor\_q command should return

   ::


       -- Schedd:  : <10.64.xx.yyy:zzzzz>
       ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD               
       0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
               

   and the condor\_status command output should be empty (no nodes
   running)

   ::



               

   Then start the elastiq service

   ::

       # service elastiq start
               

   In some minutes the minimum number of nodes should connect to the
   condor cluster and the condor\_status command output should show them
   e.g.:

   ::


       Name               OpSys      Arch   State     Activity LoadAv Mem   ActvtyTime
       slot1@10-64-22-215 LINUX      X86_64 Unclaimed Idle      0.000 1896  0+00:24:46
       slot2@10-64-22-215 LINUX      X86_64 Unclaimed Idle      0.000 1896  0+00:25:05
       slot1@10-64-22-217 LINUX      X86_64 Unclaimed Idle      0.000 1896  0+00:24:44
       slot2@10-64-22-217 LINUX      X86_64 Unclaimed Idle      0.000 1896  0+00:25:05
       slot1@10-64-22-89. LINUX      X86_64 Unclaimed Benchmar  1.000 1896  0+00:00:04
       slot2@10-64-22-89. LINUX      X86_64 Unclaimed Idle      0.040 1896  0+00:00:05
       Machines Owner Claimed Unclaimed Matched Preempting
       X86_64/LINUX        6     0       0         6       0          0
       Total               6     0       0         6       0          0
               

Troubleshooting the elastic cluster
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Check the log files**


The elastiq log file is /var/log/elastiq/elastiq.log. It is not
straightforward to interpret it without knowing its structure. When you
start the elastiq service the first part of log file reports the check
of cloud user's credentials and of other parameters configured in the
elastiq.conf file (i.e. the userdata file for slave nodes)

::

    INFO [__init__.conf] Configuration: ec2.image_id = ami-9f3da3fc (from file)
    INFO [__init__.conf] Configuration: ec2.flavour = cldareapd.medium (from file)
    INFO [__init__.conf] Configuration: ec2.api_url = https://cloud-areapd.pd.infn.it:8788/services/Cloud (from file)
    2INFO [__init__.conf] Configuration: ec2.aws_secret_access_key = [...] (from file)
    INFO [__init__.conf] Configuration: ec2.key_name = my_key (from file)
    INFO [__init__.conf] Configuration: ec2.user_data_b64 = [...] (from file)
    INFO [__init__.conf] Configuration: ec2.aws_access_key_id = [...] (from file)

    INFO [__init__.conf] Configuration: quota.max_vms = 3.0 (from file)
    INFO [__init__.conf] Configuration: quota.min_vms = 1.0 (from file)
    NFO [__init__.main] Loaded batch plugin "htcondor"
    DEBUG [htcondor.init] HTCondor plugin initialized
    DEBUG [__init__.main] EC2 image "ami-9f3da3fc" found
          

if your credentials are wrong, you will see in the log file an error as:

::

    ERROR [__init__.ec2_running_instances] Can't get list of EC2 instances (maybe wrong credentials?)
          

If you specified a wrong ami\_id for the image of slave nodes, the error
message in the log will be:

::

    ERROR [__init__.main] Cannot find EC2 image "ami-00000000"
          

Elastiq periodically checks all the VMs. If a VM is correctly added to
the condor cluster, it logs:

::

    DEBUG [__init__.ec2_running_instances] Found IP 10.64.22.188 corresponding to instance
          

otherwise something like:

::

    WARNING [__init__.ec2_running_instances] Cannot find instance 10.64.22.216 in the list of known IPs
    WARNING [__init__.ec2_running_instances] Cannot find instance 10.64.22.182 in the list of known IPs
    WARNING [__init__.ec2_running_instances] Cannot find instance 10.64.22.236 in the list of known IPs
          

When elastiq instantiates a new VM it logs something like:

::

    WARNING [__init__.ec2_scale_up] Quota enabled: requesting 1 (out of desired 1) VMs
    INFO [__init__.ec2_scale_up] VM launched OK. Requested: 1/1 | Success: 1 | Failed: 0 | ID: i-f026f340
    DEBUG [__init__.save_owned_instances] Saved list of owned instances: i-f026f340
          

and when elastiq deletes an idle VM it logs:

::

    INFO [__init__.check_vms] Host 10-64-22-190.INFN-PD is idle for more than 2400s: requesting shutdown
    INFO [__init__.ec2_scale_down] Requesting shutdown of 1 VMs...
          

In the master node of condor, logs are located in /var/log/condor
directory. You may refer to the HTCondor documentation for more
information on these log files.

::

    # ls -l /var/log/condor/
    total 76
    -rw-r--r--. 1 condor condor 24371 Jan 18 08:42 CollectorLog
    -rw-r--r--. 1 root   root     652 Jan 18 08:35 KernelTuning.log
    -rw-r--r--. 1 condor condor  2262 Jan 18 08:35 MasterLog
    -rw-r--r--. 1 condor condor     0 Jan 18 08:35 MatchLog
    -rw-r--r--. 1 condor condor 19126 Jan 18 08:42 NegotiatorLog
    -rw-r--r--. 1 root   root   13869 Jan 18 08:42 ProcLog
    -rw-r--r--. 1 condor condor   474 Jan 18 08:35 ScheddRestartReport
    -rw-r--r--. 1 condor condor  2975 Jan 18 08:40 SchedLog
          

**Check the running processes**

Processes expected to run are:

-  for the condor service:

   ::

       # ps -ef | grep condor
       condor       764       1  0 14:09 ?        00:00:00 /usr/sbin/condor_master -f
       root         960     764  0 14:09 ?        00:00:00 condor_procd -A /var/run/condor/procd_pipe -L /var/log/condor/ProcLog -R 1000000 -S 60 -C 996
       condor       961     764  0 14:09 ?        00:00:00 condor_collector -f
       condor       974     764  0 14:09 ?        00:00:00 condor_negotiator -f
       condor       975     764  0 14:09 ?        00:00:00 condor_schedd -f      
               

-  for the elastiq service:

   ::

       # ps -ef | grep elastiq
       elastiq      899       1  0 14:09 ?        00:00:00 SCREEN -dmS __|elastiq|__ /bin/sh -c python /usr/bin/elastiq-real.py --logdir=/var/log/elastiq --config=/etc/elastiq.conf --statefile=/var/lib/elastiq/state 2> /var/log/elastiq/elastiq.err
       elastiq      952     899  0 14:09 pts/0    00:00:00 /bin/sh -c python /usr/bin/elastiq-real.py --logdir=/var/log/elastiq --config=/etc/elastiq.conf --statefile=/var/lib/elastiq/state 2> /var/log/elastiq/elastiq.err
       elastiq      953     952  0 14:09 pts/0    00:00:01 python /usr/bin/elastiq-real.py --logdir=/var/log/elastiq --config=/etc/elastiq.conf --statefile=/var/lib/elastiq/state
               

.. WARNING ::
    The condor\_status information isn't updated very frequently. So it
    can happen that condor\_status shows nodes that have been already
    removed from the cloud by elastiq.
