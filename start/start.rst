.. _cicd_environment_start:

---------------
Getting Started
---------------

While the first three lab tracks highlight the benefits of Nutanix Clusters on AWS in terms of ease of lift and shift, disaster recovery, and infrastructure elasticity, this lab begins to look at the alternate path for customers looking to enable a path to multi-cloud through re-platforming applications.

In this lab track you will go through the process of migrating a simple application service, your **Fiesta** web front end, to a Docker container. You will deploy additional tools to build a Continuous Integration/Continuous Deployment (CI/CD) pipeline that will allow you define and deliver your application as code, also referred to as **Infrastructure as Code**.

The lab is designed to provide you with practical experience around the complexities and benefits of re-platforming applications. This leads into the following lab track, **Cloud Native Apps on Nutanix**, which looks at the infrastructure components of running containerized apps in Kubernetes on Nutanix.

Lab Prerequisites
+++++++++++++++++

In order to complete the following lab, you will require a **Docker Hub** account. **Docker Hub** is a web service used to create, share, manage, and deliver Docker containers within specific teams or the larger Docker community. In the lab it will be used to save and host the container image you will create.

#. Sign up for a free **Docker Hub** account here: https://hub.docker.com/

   .. figure:: images/1.png

   .. note::

      If you have an existing Docker Hub account, it may still be a good idea to create an account using an alternate e-mail address to complete the lab. As the development Docker environment will not make use of a credential store solution as you would typically see in a production environment, **your Docker Hub username and password will be stored in plaintext inside of your Docker VM**.

#. Under **Choose a Plan**, click **Continue with Free**.

#. Open the verification e-mail from Docker and click **Verify email address**.

   .. note::

      It is important to remember/store your Docker Hub username and password as they will be used in a later exercise.

Your Environment
++++++++++++++++

To let you experience the most fun and interesting parts of the lab, as well as accommodate the large number of simultaneous users, multiple components have already been staged for you. *Let's explore!*

.. .. raw:: html

   <br><center><img src="https://github.com/nutanixworkshops/gts21/raw/master/snow/gettingstarted/images/env.png"><br><i>vGTS 2021 CICD Lab Environment</i></center><br>

Docker and Fiesta VMs
.....................

Using a Calm Blueprint, each on-premises (HPOC) cluster has been pre-staged with the following VMs for each **USER**:

   .. figure:: images/2.png

   - **USER**\ *##*\ **-Fiesta_App_VM**

      A CentOS 7 VM running a NodeJS-based web application used to access the Fiesta database. The Fiesta app is a simple example of a web application for performing inventory management for party goods supply stores.

      You can validate your Fiesta application is capable of reaching your source database by browsing to \http://*USER##-Fiesta_App_VM-IP-ADDRESS*\ :5001/

   - **USER**\ *##*\ **-MariaDB_VM**

      A CentOS VM running MariaDB, which contains the **Fiesta** database providing app data for **USER**\ *##*\ **-Fiesta_App_VM**. In addition to deploying the database, the Blueprint also registers this database VM with your cluster's Era server.

   - **User**\ *##*\ **-docker_VM**

      A CentOS VM with the Docker runtime pre-installed. This will be used to create and test your containerized application.

Tools VM
........

The Windows Tools VM provides a number of different applications used across multiple Nutanix Bootcamps. Filter for your **USER**\ *##* in **Prism Central > Virtual Infrastructure > VMs** and take note of the IP of your **USER**\ *##*\ **WinToolsVM** VM, as you will be connecting to this VM via RDP (using **NTNXLAB** Administrator credentials).

   .. figure:: images/3.png

In the following labs you will use the Tools VM to access the **Visual Studio Code** text editor, which has been pre-installed with a number of extensions for working with Git and containers.
