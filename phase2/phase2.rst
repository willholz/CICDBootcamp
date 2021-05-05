.. _phase2_container:

---------------------------------
Building the CI/CD Infrastructure
---------------------------------

In software engineering, CI/CD or CICD generally refers to the combined practices of continuous integration and either continuous delivery or continuous deployment. CI/CD bridges the gaps between development and operation activities and teams by enforcing automation in building, testing and deployment of applications.

There are `multiple CI/CD platforms <https://www.katalon.com/resources-center/blog/ci-cd-tools/>`_, including popular solutions like Jenkins. In this exercise we will deploy a platform called **Drone** due to its simplicity of deployment and basic use.

In addition to the CI platform, we will also require a supported version control manager to host our Fiesta source code. GitHub and GitLab are common, cloud-hosted solutions you would expect to see in many Enterprise environments. For the sake of providing a streamlined, self-contained lab, you will deploy an instance of **Gitea**. **Gitea** is a lightweight, open-source solution for self-hosting Git, with an interface similar to GitHub.

But first - your most important tool is your development environment!

In creating the initial containerized version of Fiesta, we used a command line text editor (ex. **vi** or **nano**) to manipulate files. While these tools can certainly do the job, as we've seen, this method is not exactly easy, or efficient to modify files on a large scale.

In this exercise, we'll graduate to **Visual Studio Code**. **Visual Studio Code** is a free source-code editor made by Microsoft for Windows, Linux and macOS. Features include support for debugging, syntax highlighting, intelligent code completion, snippets, code refactoring, and embedded Git.

Visual Studio Code (VSC)
++++++++++++++++++++++++

#. Connect to your **USER**\ *##*\ **-WinTools** VM via an RDP client using the **NTNXLAB\\Administrator** credentials.

   .. note::

      Refer to :ref:`clusterdetails` for Active Directory username and password information.

#. From the desktop, open **Tools > Visual Studio Code**.

#. Click **View > Command Palette...**.

   .. figure:: images/1.png

#. Type **Remote SSH**, and select **Remote-SSH: Connect Current Window to Host...**.

   .. figure:: images/2.png

#. Click on **+ Add New SSH Host...** and type **ssh root@**\ *<User##-docker_VM-IP-ADDRESS>* and hit **Enter**.

   .. figure:: images/2b.png

#. Select the location **C:\\Users\\Administrator\ \\.ssh\\config** (typically first entry) to update the config file.

#. Select **Connect** on the pop-up in the bottom right corner to connect to the VM.

   .. note::

      If you miss this dialog box:

      - Click **View > Command Palette...**
      - Type **Remote-SSH** and select **Remote-SSH: Connect to Host**
      - Select the **User**\ *##*\ **-docker_VM** IP

#. A new Visual Studio Code window will open. In the **Command Palette** make the following selections:

   - **Select the platform of the remote host** - Linux
   - **Are you sure you want to continue?** - Continue
   - **Password** - nutanix/4u

#. Press **Enter** to connect to the remote host.

   .. note::

      You can disregard the messages in the lower right-hand corner by clicking **Don't Show Again**.

      .. figure:: images/3.png

#. Click the **Explorer** button from the left-hand toolbar and select **Open Folder**.

   .. figure:: images/4.png

#. Provide the ``/`` as the folder you want to open and click on **OK**.

   Ensure that **bin** is NOT highlighted otherwise the editor will attempt to autofill ``/bin/``. You can avoid this by clicking in the path field *before* clicking **OK**.

   .. figure:: images/4b.png

#. If prompted, provide the password again and press **Enter**.

   The initial connection may take up to 1 minute to display the root folder structure of the **User**\ *##*\ **-docker_VM** VM.

   .. note::

      You can disregard the warning regarding **Unable to watch for file changes in this large workspace folder.**

#. Once the folder structure appears, open **/root/github**. You should see the cloned **Fiesta** repository, your **dockerfile** and **runapp.sh**.

   .. figure:: images/5.png

   Having a rich text editor capable of integrating with the rest of our tools, and providing markup to the different source code file types will provide significant value in upcoming exercises and is a much simpler experience for most users compared to command line text editors.

Deploying Gitea
+++++++++++++++

In this exercise we will deploy **Gitea** and its required **MySQL** database as containers running on your Docker VM using a **YAML** file and the ``docker compose`` command.

#. In **Virtual Studio Code**, select **Terminal > New Terminal** from the toolbar.

   .. figure:: images/6.png

   This will open a new SSH session to your **User**\ *##*\ **-docker_VM** VM using a terminal built into the text editor - *convenient!*

   .. note::

      You can also use your preferred SSH client to connect to **User**\ *##*\ **-docker_VM**. Using the **Virtual Studio Code** terminal is not a hard requirement.

#. You can expand the terminal window by clicking the **Maximize Panel Size** icon as shown below.

   .. figure:: images/6b.png

#. In the terminal, run the following commands to create the directories required for the deployment:

   .. code-block:: bash

       mkdir -p ~/github
       mkdir -p /docker-location/gitea
       mkdir -p /docker-location/drone/server
       mkdir -p /docker-location/drone/agent
       mkdir -p /docker-location/mysql

#. Run ``cd ~/github``.

#. Run ``curl --silent https://raw.githubusercontent.com/nutanixworkshops/gts21/master/cicd/docker_files/docker-compose.yaml -O`` to download the **YAML** file describing the CI/CD infrastructure.

   You can easily view the **YAML** file in **Visual Code Studio** by selecting and refreshing your **/github/** directory and selecting the **docker-compose.yaml** file.

   .. figure:: images/8b.png

#. Run ``docker login`` and provide the credentials for your Docker Hub account created during :ref:`environment_start`.

   .. note::

      If you opened the file in the previous step, you can click the **Maximize** icon in your Terminal session again to restore it to full screen.

#. Run ``docker-compose create db gitea`` to build the **MySQL** and **Gitea** containers.

   When returns you should see that the two services have been created, similar to below.

   .. figure:: images/9.png

#. Run ``docker-compose start db gitea`` to start the **MySQL** and **Gitea** containers.

Configuring Gitea
+++++++++++++++++

In order to use Gitea for authentication within Drone, which will be configued in a later step, Gitea must be configured to use **HTTPS**. As this is a lab environment, we will configure Gitea to use a self-signed SSL certificate.

To do so we will use ``docker exec`` to execute commands *within* the Gitea container.

#. Run ``docker exec -it gitea /bin/bash`` to access the Gitea container shell.

#. From the container's **bash** prompt, run ``gitea cert --host <IP ADDRESS OF THE DOCKER VM>``.

   This will create two files **cert.pem** and **key.pem** in the root of the container.

   .. figure:: images/10.png

#. Copy the \*.pem files by running ``cp /*.pem /data/gitea``

#. Run ``chmod 744 /data/gitea/*.pem``

#. Close the container shell by pressing **CTRL+D**

#. Open a browser and point it to **http://<IP ADDRESS DOCKER VM>:3000**

   .. note::

      The WinToolsVM has Google Chrome pre-installed.

#. Make the following changes to the default **Initial Configuration**:

   - Under **Database Settings**

     - **Host** - *<IP ADDRESS OF YOUR DOCKER VM>*:3306
     - **Password** - gitea

   .. figure:: images/10-1.png

   - Under **General Settings**

      .. note::

         Ensure you are updating the **Base URL** from **HTTP** to **HTTPS**!

     - **SSH Server Port**: 2222
     - **Gitea Base URL**: **https**://*<IP ADDRESS OF YOUR DOCKER VM>*:3000

   .. figure:: images/11.png

#. Click **Install Gitea** at the bottom of the page.

   You should receive an error indicating **This site can’t provide a secure connection**, which we will fix using the self-signed SSL certificate previously created.

#. Return to your existing **Visual Studio Code** session.

#. From the **Explorer** side panel, open **/docker-location/gitea/conf/app.ini**.

#. Add the following lines under the **[server]** section as shown in the image below:

   .. code-block:: ini

       PROTOCOL = https
       CERT_FILE = cert.pem
       KEY_FILE = key.pem

   .. figure:: images/12.png

#. Save the file.

#. From your terminal session, restart the container by running ``docker-compose restart gitea``.

#. Reload the browser (\https://*<IP ADDRESS OF YOUR DOCKER VM>*:3000).

   .. figure:: images/12b.png

   You should now receive a typical certificate error, which is expected using a self-signed certificate. Proceed to the login page (ex. Click **Advanced > Proceed to...**).

#. Click **Need an account? Register now.** to create the initial user account.

   By default, the first user account created will have full administrative priveleges within the Gitea application.

#. Fill out the following:

   - **Username** - nutanix
   - **Email Address** - nutanix@nutanix.com
   - **Password** - nutanix/4u

#. Click **Register Account**.

   .. figure:: images/14b.png

   You now have a self-hosted Git repository running inside of your Docker development environment as a container. The final step is to deploy and configure Drone.

Deploying Drone
+++++++++++++++

You may have noticed that the **Drone** service is described in the same **docker-compose.yaml** file as **Gitea** and its **MySQL** database service, yet we did not deploy it in the previous exercise. This is because we first need to update the **Drone** service **docker-compose.yaml** with some additional information from the **Gitea** deployment in order for **Drone** to use **Gitea** as a source for OAuth authentication services.

#. In **Gitea** (\https://*<IP ADDRESS OF YOUR DOCKER VM>*:3000), click the icon in the upper right-hand corner and select **Settings** from the dropdown menu.

   .. figure:: images/15.png

#. Select **Applications**.

#. Under **Manage OAuth2 Applications > Create a new OAtuh2 Application**, fill out the following:

   - **Application Name** - drone
   - **Redirect URI** - http://*<DOCKER-VM-IP-ADDRESS>*:8080/login

   .. figure:: images/15b.png

#. Click the **Create Application** button.

#. On the following screen, copy the **Client ID** and the **Client Secret** to a text file (ex. **Notepad**), as you will need both values in the following steps.

   .. figure:: images/16b.png

#. Click **Save**.

#. Return to your existing **Visual Studio Code** session.

#. From the **Explorer** side panel, open **/root/github/docker-compose.yaml**.

#. Under **drone-server > environment**, update the following fields:

   - **DRONE_GITEA_SERVER** - \https://*<IP ADDRESS OF DOCKER VM>*:3000
   - **DRONE_GITEA_CLIENT_ID** - *Client ID from Gitea*
   - **DRONE_GITEA_CLIENT_SECRET** - *Client Secret from Gitea*
   - **DRONE_SERVER_HOST** - *<IP ADDRESS OF DOCKER VM>*:8080

   .. figure:: images/17b.png

#. Under **drone-docker-runner > environment**, update the following fields:

   - **DRONE_RPC_HOST** - *<IP ADDRESS OF DOCKER VM>*:8080

   .. figure:: images/18b.png

#. Save **docker-compose.yaml**.

#. Return to your Terminal session.

#. Run ``docker-compose create drone-server drone-docker-runner`` to build the **Drone** containers.

#. Run ``docker-compose start drone-server drone-docker-runner`` to start **Drone**.

#. Open ``http://<DOCKER-VM-IP-ADDRESS>:8080`` in a new browser tab.

   .. note::

      This will try to authenticate the **nutanix** user defined as **DRONE_USER_CREATE** in the **docker-compose.yaml** file.

#. When prompted, click **Authorize Application**.

   .. figure:: images/19.png

#. You should be presented with the **Drone** UI, which will not yet have any source code repositories listed.

   .. figure:: images/18.png

.. raw:: html

    <H1><font color="#B0D235"><center>Congratulations!</center></font></H1>

You have successfully provisioned all the infrastructure for your CI/CD pipeline, **but** there is still more to be done:

- **Visual Studio Code** is a big usability upgrade over **vi** :fa:`thumbs-up`
- We still need to automate our container building, testing, and deployment :fa:`thumbs-down`
- The image is only available as long as the Docker VM exists :fa:`thumbs-down`
- The start of the container takes a long time :fa:`thumbs-down`

The following labs will address our :fa:`thumbs-down` issues - Let's go for it! :fa:`thumbs-up`
