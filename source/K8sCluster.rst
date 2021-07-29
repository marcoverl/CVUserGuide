Creating Kubernetes Clusters on the Cloud
=========================================
.. _CreatingK8s:


The virtual machines provided by CloudVeneto can also be used to deploy
Kubernetes clusters where users familiar with Kubernetes can run their
applications. `Kubernetes (K8S) <https://kubernetes.io/>`__ is the most
popular production-grade container orchestrator.

In this chapter we explain how to deploy a Kubernetes cluster using
`Ansible <https://www.ansible.com/>`__.

Intro: the idea
---------------

The provided Ansible playbooks allow you to deploy a Kubernetes cluster
both on bare metal and on an OpenStack cloud. The installation is based
on the *kubeadm* tool configured with a pre-generated admin token and
flannel network. The playbooks can enrich the cluster installation with
a set of services such as:

-  Dashboards: K8S legacy and Grafana;

-  Monitoring: Prometheus operator;

-  Big Data Analytics: Apache Spark and Kafka operators.

System requirements
-------------------

The deployment environment requires:

-  Ansible 2.5.0+ (on the user client)

-  Ubuntu 18.04 (for master and node images)
   These Ansible playbooks have only been tested on Ubuntu 18.04, so we do not guarantee the correct Kubernetes deployment on different Ubuntu versions or Linux distributions.

-  2 CPUs or more per machine, 2 GB or more of RAM per machine (any less will leave little room for your apps)

Getting Started
---------------

This section represents a quick installation and is not intended to
teach you about all the components. The easiest way to get started is to
clone the 'ansible-k8s' repository:

::

    # git clone https://github.com/zangrand/ansible-k8s.git
    # cd ansible-k8s
          

The directory structure should be like:

::

    ansible-k8s/
    +-- config
    |   +-- config
    |   +-- keystone_client.py
    |   +-- tls-ca-bundle.pem
    +-- examples
    |   +-- spark-pi.yaml
    |   +-- kcluster.yaml
    |   +-- ktopic.yaml
    +-- deploy_k8s.yaml
    +-- deploy_master_openstack.yaml
    +-- deploy_node_openstack.yaml
    +-- group_vars
    |   +-- all
    +-- inventory
    +-- k8s
    |   +-- alertmanager-service.yaml
    |   +-- dashboard-setup.yaml
    |   +-- grafana-service.yaml
    |   +-- os-k8s-node.yaml
    |   +-- prometheus-service.yaml
    +-- openstack_config.yaml
    +-- README.md
    +-- roles
        +-- auth
        |   +-- keystone
        |       +-- files
        |       |   +-- infn_ca.pem
        |       |   +-- k8s-auth-policy.yaml
        |       |   +-- k8s-keystone-auth.yaml
        |       |   +-- k8s-keystone-auth.yaml_orig
        |       |   +-- keystone_client.py
        |       |   +-- tls-ca-bundle.pem
        |       |   +-- webhookconfig.yaml
        |       +-- tasks
        |           +-- main.yml
        +-- common
        |   +-- tasks
        |       +-- main.yml
        +-- docker
        |   +-- tasks
        |       +-- main.yml
        +-- haproxy
        |   +-- tasks
        |   |   +-- main.yml
        |   +-- templates
        |       +-- haproxy.cfg.j2
        +-- keepalived
        |   +-- tasks
        |       +-- main.yml
        |   +-- templates
        |       +-- keepalived.conf.j2
        +-- kubeadm
        |   +-- tasks
        |       +-- main.yml
        +-- master
        |   +-- handlers
        |   |   +-- main.yml
        |   +-- tasks
        |       +-- main.yml
        +-- masterha
        |   +-- handlers
        |   |   +-- main.yml
        |   +-- tasks
        |       +-- main.yml
        |   +-- templates
        |       +-- kubeadm-config.yaml.j2
        +-- node
        |   +-- tasks
        |      +-- main.yml
        +-- os-node
        |  +-- tasks
        |      +-- main.yml
        +-- prometheus
        |   +-- tasks
        |       +-- main.yml
        +-- spark
            +-- tasks
                +-- main.yml
          

Deployment on the cloud
-----------------------

The provided Ansible playbook is able to create and configure properly
all hosts (i.e. the VMs) on CloudVeneto and deploy Kubernetes on it. To
do it:

-  edit the file openstack\_config.yaml and fill up all required attributes 
   (i.e. OS\_USERNAME, OS\_PASSWORD, OS\_PROJECT\_NAME, OS\_PROJECT\_ID, OS\_NETWORK,
   etc), the same used for accessing OpenStack by its client;

-  define the VMs characteristics of the master and nodes, in terms of
   name, flavor, and image (default values for flavor and image are defined);

-  specify the number of nodes (i.e. OS\_NODES) of your cluster.

Look at the comments inside the openstack\_config.yaml file for more details. 

Verify if the 'shade' Python module is available on your environment,
otherwise install it:
::

    # pip install shade
          

Execute:
::
    
    # export ANSIBLE_HOST_KEY_CHECKING=False
    # ansible-playbook deploy_master_openstack.yaml
          

.. NOTE ::
    The deployment requires a few minutes to have the full cluster up
    and running.

How to access your Kubernetes cluster
-------------------------------------


There are two different ways to access the Kubernetes cluster: the
*kubectl* command line tool or the dashboard.

Kubectl
^^^^^^^

The kubectl command line tool is available on the master node. If you
wish to access the cluster remotely please see the following guide:
`Install and Set Up
kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`__.

You can enable your local kubectl to access the cluster through the
Keystone authentication. To do it, copy all files contained into the
folder ansible-k8s/config/ to $HOME/.kube/.

The tls-ca-bundle.pem file is CA certificate required by the CloudVeneto
OpenStack based cloud. Do not forget to source the openrc.sh with your
OpenStack credentials and OS\_CACERT variable set.

The only manual configuration required is to edit $HOME/.kube/config and set the IP address of your new K8S master.

To allow other users to access your K8S cluster and operate on a subset of its resources, edit the auth-policy file with:
::

    # kubectl -n kube-system edit configmap k8s-auth-policy

modify in the first block the line "resources" and replace "type": "role", "values": ["k8s-user"] with e.g. "type": "user", "values": ["username1", "username2"]

Kubernetes Dashboard
^^^^^^^^^^^^^^^^^^^^

The cluster exposes the following dashboards:

-  K8S dashboard: https://master\_ip:30900

-  Prometheus UI: http://master\_ip:30901

-  Alertmanager UI: http://master\_ip:30902

-  Grafana UI: http://master\_ip:30903

To login into the K8S dashboard use the token of the kube-system:default
service account. To get it, execute the following command from your
environment, or from the master node:

::

    # kubectl -n kube-system describe secret kubernetes-dashboard
    [...]
    Type:  kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     1025 bytes
    namespace:  11 bytes
    token:      eyJhbGciOiJSUzI1NiIsImtpZZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3Vu
    dC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWU
    iOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi05cGc5NyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2
    VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY
    291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjAyYjYzZGRkLWFhYzItMTFlOC1iNTBkLWZhMTYzZTQ2OWU0ZiIsInN1
    YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.XPSzu31Svk
    R_xwfd3MpLQBkHc7anZlEA1FMSMrZsU6wENflLJQEPrEUJmYji24jU4vTnd2eVK1rhEB4P1iPEiVg0nCZIIhkJTtpaN
    TyefV1Uq3V9JUTxEO9rMAsfSx16yqctuSi9qgUU7Ac85ZEffJqrKrQwSkQGyCnrDuAQ11Ryl5VGWbTfTfeEP-epjm0j
    nAcI1akhkoS2xUESRV9Bq41rOtboJYv3hAe0pjOL7CHZ3mTsHMHXR_0IDQvCTx8tC9S_vU09-jK8c_4UAkoUDd5-_1D
    Pl68AckAMtgZyPSQLKnlFW50WwQt5WCwp7VGrBL_okM-E7QeTQkrUMrGTDw
            

.. NOTE ::
    To login into the Grafana dashboard as administrator use the
    credentials: username=admin and password=admin. The first login
    requires the changing of the default password for security reasons.

Testing your Kubernetes cluster
-------------------------------

The cluster comes up by default with two K8S operators implementing the
popular Big Data Analytics and Streaming platforms Apache
`Spark <https://spark.apache.org/>`__ and
`Kafka <https://kafka.apache.org/>`__ (you can avoid this by removing
the roles spark and kafka in the file deploy\_k8s.yaml).

Launching the Spark application spark-pi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use the Spark application spark-pi to verify that the cluster
works properly. Just take the examples/spark-pi.yaml file and execute
the following kubectl commands:

::

                                                                                                     
    # kubectl apply -f spark-pi.yaml

    # kubectl get sparkapplications spark-pi
    NAME       AGE
    spark-pi   5m

    # kubectl describe sparkapplications spark-pi
    Name:         spark-pi
    Namespace:    default
    Labels:       none
    Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                    {"apiVersion":"sparkoperator.k8s.io/v1beta1","kind":"SparkApplication","metadata":{"annotations":{},"name":"spark-pi","namespace":"default...
    API Version:  sparkoperator.k8s.io/v1beta1
    Kind:         SparkApplication
    [...]

    # kubectl logs -f spark-pi-driver | grep "Pi is roughly"
    Pi is roughly 3.1458557292786464

            

.. NOTE ::
    In case of problems with the sparkoperator API Version, look at the
    output of

    ::

        # kubectl api-versions

Creating a Kafka cluster with a topic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Declare the cluster structure as in the kcluster.yaml file taken from the examples directory, and execute the following kubectl command: 
::

    # kubectl apply -f kcluster.yaml

For further details on configuration see https://strimzi.io/docs/master/#assembly-deployment-configuration-str

A topic for the Kafka cluster can be declared as in the ktopic.yaml file taken from the examples directory, and created by executing the following kubectl command:
::
    # kubectl apply -f ktopic.yaml

Kubernetes provides a port on the master for accessing the created cluster.
The port number is reported by the following kubectl command:
::

    # kubectl get service kcluster-kafka-external-bootstrap -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

Other useful commands for monitor the status of the cluster are:
::

    # kubectl get service
    NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    kcluster-kafka-0                        NodePort    10.97.1.118      <none>        9094:31945/TCP               64s
    kcluster-kafka-1                        NodePort    10.100.252.199   <none>        9094:31730/TCP               64s
    kcluster-kafka-2                        NodePort    10.106.128.149   <none>        9094:31608/TCP               64s
    kcluster-kafka-bootstrap                ClusterIP   10.109.113.86    <none>        9091/TCP                     65s
    kcluster-kafka-brokers                  ClusterIP   None             <none>        9091/TCP                     65s
    kcluster-kafka-external-bootstrap       NodePort    10.107.133.0     <none>        9094:32161/TCP               64s
    kcluster-zookeeper-client               ClusterIP   10.103.223.73    <none>        2181/TCP                     93s
    kcluster-zookeeper-nodes                ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP   93s
    kubernetes                              ClusterIP   10.96.0.1        <none>        443/TCP                      3d1h
    
    # kubectl get pod 
    NAME                                            READY   STATUS    RESTARTS   AGE
    kcluster-entity-operator-7b8d767b5c-lh6kp       3/3     Running   0          3m55s
    kcluster-kafka-0                                2/2     Running   0          4m28s
    kcluster-kafka-1                                2/2     Running   0          4m28s
    kcluster-kafka-2                                2/2     Running   0          4m28s
    kcluster-zookeeper-0                            2/2     Running   0          4m56s
    kcluster-zookeeper-1                            2/2     Running   0          4m56s
    kcluster-zookeeper-2                            2/2     Running   0          4m56s
    strimzi-cluster-operator-6464cfd94f-tmbqd       1/1     Running   0          3d1h

    # kubectl get kafkatopics
    NAME                  AGE
    ktopic                12s

