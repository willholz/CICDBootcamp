.. title:: Nutanix Containerizing Apps & CI/CD Bootcamp

.. toctree::
   :maxdepth: 2
   :name: _cicd
   :hidden:

   gettingstarted/gettingstarted
   basic/basic
   phase2/phase2
   phase3/phase3
   phase4/phase4
   phase5/phase5

---------------------------------
Containerizing Apps & CI/CD
---------------------------------

While Nutanix offers simple paths to a multi-cloud environment using Nutanix Clusters on AWS, providing easy lift and shift, disaster recovery, and infrastructure elasticity, this lab begins to look at the alternate path for customers looking to enable a path to multi-cloud through re-platforming applications.

In this Bootcamp you will go through the process of migrating a simple application service, your **Fiesta** web front end, to a Docker container. You will deploy additional tools to build a Continuous Integration/Continuous Deployment (CI/CD) pipeline that will allow you define and deliver your application as code, also referred to as **Infrastructure as Code**.

The lab is designed to provide you with practical experience around the complexities and benefits of re-platforming applications. This leads into another Bootcamp, **Getting Started with Kubernetes & Nutanix Karbon**, which looks at the infrastructure components of running containerized apps in Kubernetes on Nutanix.


Agenda
++++++

- Introductions
- Lab Setup

Introductions
+++++++++++++

- Name
- Familiarity with Nutanix?

Initial Setup
+++++++++++++

- Take note of the *Passwords* being used
- Log into your virtual desktops (connection info below)

Cluster assignment
++++++++++++++++++

The instructor will tell the attendees their assigned UserXX

.. note::
  If these are Single Node Clusters (SNCs) pay close attention to networking. The SNCs are setup and configured differently compared to the three/four node clusters. Details are within the different cluster sections directly below.

Environment Details
+++++++++++++++++++

Nutanix Workshops are intended to be run in the Nutanix Hosted POC environment. Your cluster will be provisioned with all necessary images, networks, and VMs required to complete the exercises.

Networking
..........

As both three/four node clusters and single node clusters are available in the HPOC environment, we will outline the details of each separately.

Three/Four node HPOC clusters
-----------------------------

Three or four node Hosted POC clusters follow a standard naming convention:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.\ **38**\ .\ *XYZ*\ .0
- **Cluster IP** - 10.\ **38**\ .\ *XYZ*\ .37

For example:

- **Cluster Name** - POC055
- **Subnet** - 10.38.55.0
- **Cluster IP** - 10.21.55.37 for the VIP of the Cluster

Throughout the Workshop there are multiple instances where you will need to substitute *XYZ* with the correct octet for your subnet, for example:

.. list-table::
  :widths: 25 75
  :header-rows: 1

  * - IP Address
    - Description
  * - 10.38.\ *XYZ*\ .37
    - Nutanix Cluster Virtual IP
  * - 10.38.\ *XYZ*\ .39
    - **PC** VM IP, Prism Central
  * - 10.38.\ *XYZ*\ .41
    - **DC** VM IP, NTNXLAB.local Domain Controller

Each cluster is configured with 2 VLANs which can be used for VMs:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.38.\ *XYZ*\ .1/25
    - 0
    - 10.38.\ *XYZ*\ .50-10.38.\ *XYZ*\ .124
  * - Secondary
    - 10.38.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.38.\ *XYZ*\ .132-10.38.\ *XYZ*\ .253

Single Node HPOC Clusters
-------------------------

For some workshops we are using Single Node Clusters (SNCs). The reason for this is to allow more people to have a dedicated cluster, but still have enough free clusters for the larger workshops, including those for customers.

The network in the SNC config is using a /26 network. This splits the network address into four equal sizes that can be used for workshops. The below table describes the setup of the network in the four partitions. It provides essential information for the workshop with respect to the IP addresses and the service running at that IP address.

.. list-table::
  :widths: 15 15 15 15 40
  :header-rows: 1

  * - Partition 1
    - Partition 2
    - Partition 3
    - Partition 4
    - Service
    - Comment
  * - 10.38.x.1
    - 10.38.x.65
    - 10.38.x.129
    - 10.38.x.193
    - Gateway
    -
  * - 10.38.x.5
    - 10.38.x.69
    - 10.38.x.133
    - 10.38.x.197
    - AHV Host
    -
  * - 10.38.x.6
    - 10.38.x.70
    - 10.38.x.134
    - 10.38.x.198
    - CVM IP
    -
  * - 10.38.x.7
    - 10.38.x.71
    - 10.38.x.135
    - 10.38.x.199
    - Cluster IP
    -
  * - 10.38.x.8
    - 10.38.x.72
    - 10.38.x.136
    - 10.38.x.200
    - Data Services IP
    -
  * - 10.38.x.9
    - 10.38.x.73
    - 10.38.x.137
    - 10.38.x.201
    - Prism Central IP
    -
  * - 10.38.x.11
    - 10.38.x.75
    - 10.38.x.139
    - 10.38.x.203
    - AutoAD IP(DC)
    -
  * - 10.38.x.32-37
    - 10.38.x.96-101
    - 10.38.x.160-165
    - 10.38.x.224-229
    - Objects 1
    -
  * - 10.38.x.38-58
    - 10.38.x.102-122
    - 10.38.x.166-186
    - 10.38.x.230-250
    - Primary network IPAM
    - 6 Free IPs free for static assignment

Credentials
...........

.. note::

  The *<Cluster Password>* is unique to each cluster and will be provided by the leader of the Workshop.

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Credential
     - Username
     - Password
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

Each cluster has a dedicated domain controller VM, **DC**, responsible for providing AD services for the **NTNXLAB.local** domain. The domain is populated with the following Users and Groups:

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Group
     - Username(s)
     - Password
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

.. _clusterdetails:

Access Instructions
+++++++++++++++++++

The Nutanix Hosted POC environment can be accessed a number of different ways:

Lab Access User Credentials
...........................

PHX Based Clusters:
**Username:** PHX-POCxxx-User01 (up to PHX-POCxxx-User20), **Password:** *<Provided by Instructor>*

RTP Based Clusters:
**Username:** RTP-POCxxx-User01 (up to RTP-POCxxx-User20), **Password:** *<Provided by Instructor>*

Frame VDI
.........

Login to: https://console.nutanix.com/x/labs

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Parallels VDI
.................

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Employee Pulse Secure VPN
..........................

Download the client:

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Install the client.

In Pulse Secure Client, **Add** a connection:

For PHX:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

For RTP:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com


Nutanix Version Info
++++++++++++++++++++

- **AOS Version** - 5.18.x | 5.19.x
- **PC Version** - Prism 2021.3
