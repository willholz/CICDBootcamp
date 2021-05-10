.. _phase3_container:

---------------------
Building The Pipeline
---------------------

In this exercise you will develop and test your CI/CD pipeline, including:

  - Build and tag the images using a versioned naming convention
  - Test the build images
  - Upload the images to Docker Hub so they persist outside your single VM development environment
  - Deploy the images as Docker containers

..
   .. note::
   Estimated time **45-60 minutes**

   Now that we have our tooling and basic CI/CD infrastructure up and running let's start using it. To do that we need to run a few steps.

   - Create a repo in Gitea
   - Tell our development environment to use the Gitea environment
   - Configure Drone to run

     - build images
     - test images
     - save images in Dockerhub
     - deploy the image as containers

Creating A Repository
+++++++++++++++++++++

First we will create a repository (or *repo* for short) that we can use to store our files in from which we want to have our images/containers build.

#. Return to **Gitea** in your browser (\https://*<DOCKER-VM-IP-ADDRESS>*:3000).

   If prompted, log in using the **nutanix** account you created during deployment.

#. Click the **+** sign in the top right-hand corner and select **+ New Repository** from the dropdown menu.

   .. figure:: images/1.png

#. Update the following field:

   - **Repository Name** - Fiesta_Application

#. Click **Create Repository** at the bottom of the page.

#. After the repo has been created, note the **HTTPS** URL **Gitea** displays under **Clone this repository**. This URL will be used in an upcoming step.

   .. figure:: images/2.png

#. In your **USER**\ *##*\ **WinToolsVM** VM, open **PowerShell**.

#. In PowerShell, run ``git config --global http.sslVerify false``

   By default, git will not allow you to clone anything from a Version Control Manager using self-signed SSL certificates.

#. Make the appropriate substitutions for your name and e-mail, and run the following two commands:

   .. code-block:: bash

       git config --global user.name "FIRST_NAME LAST_NAME"
       git config --global user.email "MY_NAME@MY_DOMAIN.com"

   .. figure:: images/2b.png

   This will identify your local source code commits when they are pushed into the remote **Gitea** repository.

#. Return to **Visual Code Studio** and click **File > New Window** to open a **Local** session.

   .. figure:: images/3.png

#. In the new **Visual Code Studio** window, click **View > Command Palette**.

#. Type ``git clone`` and press **Return**.

#. Copy and paste the **HTTPS** (not **SSH**) URL from **Step 5** found in **Gitea**.

#. Select **Clone from URL**.

   .. figure:: images/5.png

#. In the **Select Folder** window, create a new folder named **github** under your user directory.

#. Select the folder and click **Select Repository Location**.

   .. figure:: images/5b.png

   This is the local directory where files from your Gitea **Fiesta_Application** repo will be cloned.

#. When prompted **Would you like to open the cloned repository**, click **Open**.

   .. figure:: images/7.png

   .. note::

      If the dialog disappears before you're able to click **Open**, click **File > Open Folder** and select your **C:\\Users\\Administrator\\github** directory. Click **Select Folder**.

   You should now see an empty **Fiesta_Application** directory under the **Explorer** pane in **Visual Studio Code**.

   .. figure:: images/7b.png

#. Right-click the **Fiesta_Application** folder in the **Explorer** pane and select **New File**.

#. Specify **README.md** as the file name and press **Return** to create the blank file.

#. Paste the text below in **README.md** and save the file.

   .. code-block:: md

    # Fiesta Application

    This Repo is built for the Fiesta Application and has all needed files to build the containerized version of the Fiesta app.

    Original version of the Fiesta Application can be found at https://github.com/sharonpamela/Fiesta

#. As the Git extension has been pre-installed in **Visual Code Studio**, you will observe the **Source Control** icon (highlighted in the screenshot below) now has a blue **1** icon on top.

   This indicates that there is one outstanding change in the local Git repository that has not yet been committed.

   .. figure:: images/9.png

#. Click the **Source Control** icon in the left-hand toolbar.

#. Enter the description of your changes in the **Message** field (ex. *Initial README commit*) and click the :fa:`check` icon to commit your changes.

#. Select **Always**.

#. Next the **SOURCE CONTROL**, click **... > Push** to push your **README.md** commit to the repo in **Gitea**.

   .. figure:: images/9b.png

#. When prompted, provide your **Gitea** user credentials:

   - **Username** - nutanix
   - **Password** - nutanix/4u

#. Provide the login information for Gitea (user name is nutanix and password is the default password)

   .. note::

    In the lower right-hand corner you will get a prompt asking if you would like to periodically run a ``git fetch``. This is useful if you have multiple people working against the repo, but is unnecessary for the lab. Click **No** or allow the dialog to time out.

    .. figure:: images/10.png

#. Return to **Gitea** and select your **Fiesta_Application** repo from the **Dashboard**, under **Repositories**.

   .. figure:: images/11b.png

   You should now see your initial commit.

Adding Your Repo To Drone
+++++++++++++++++++++++++

Now that you have created and populated a Git repository, we can configure **Drone** to monitor for the ``git push`` and perform tasks in response.

First, **Drone** needs to understand which Git repos to track.

#. Open **Drone** in your browser (\http://*<IP ADDRESS DOCKER VM>*:8080)

   .. note::

      As **Drone** is using **Gitea** for authentication, you should not be prompted to login.

#. Click the **SYNC** button to have **Drone** add the repos associated with your **Gitea** account.

   After ~30 seconds you will see your **Fiesta_Application** repo.

#. Next to your **Fiesta_Application** repo, click **ACTIVATE**.

#. Click **ACTIVATE REPOSITORY**.

#. Under **Settings > Main > Project settings**, select the **Trusted** checkbox.

   .. figure:: images/11c.png

   This is required to allow **Drone** to use the repo.

#. Click **Save**.

Adding Tasks To Drone
+++++++++++++++++++++

Drone is looking for a file **.drone.yml** in the root of the repo to tell it what Drone has to do. The first step we'll add to **Drone** is automating building our Docker image after each code push.

#. Return to your **Visual Studio Code (Local)** window.

   .. note::

      This is the instance of **Visual Studio Code** used to create and modify your **README.md** file at the beginning of this exercise - *not* the **SSH** instance connected to your Docker VM.

#. Under **Explorer**, right-click **Fiesta_Application** and select **New File**.

#. Create a file in the root of **Fiesta_Application** named **.drone.yml**

#. Copy and paste the contents below into **.drone.yml**:

   .. code-block:: yaml

      kind: pipeline
      name: default

      clone:
        skip_verify: true

      steps:

      - name: build test image
        image: docker:latest
        pull: if-not-exists
        volumes:
          - name: docker_sock
            path: /var/run/docker.sock
        commands:
          - docker build -t fiesta_app:${DRONE_COMMIT_SHA:0:6} .

      volumes:
      - name: docker_sock
        host:
          path: /var/run/docker.sock


#. Save the file. Similar to your initial **README.md** commit, you will now push this file into your **Gitea** repo.

#. Select the **Source Control** icon from the left-hand toolbar.

   .. note::

      Again, this icon should now have a blue **1** icon indicating 1 uncommitted file. You can also hover above the toolbar icons to see their names.

#. Provide a commit message in the **Message** field and click the :fa:`check` icon to commit.

#. Next the **SOURCE CONTROL**, click **... > Push**.

   Drone has now seen a ``git push`` action and will follow the content of the **.drone.yml** file.

#. Return to **Drone** and click the **Drone** icon in the upper left-hand of the screen to return to the dashboard.

#. Select **nutanix/Fiesta_Application > ACTIVITY FEED > #1 > build test image** and note the errors.

   .. figure:: images/12.png

   The build was searching for a dockerfile, but couldn't find it. *Let's fix that!*

#. Return to your **Visual Studio Code (Local)** window.

#. Create a new file in the root of the **Fiesta_Application** named **dockerfile** and paste the content below into the file.

   .. code-block:: docker

      # Grab the needed OS image
      FROM wessenstam/ntnx-alpine:latest

      # Install the needed packages
      RUN apk add --no-cache --update nodejs npm mysql-client git python3 python3-dev gcc g++ unixodbc-dev curl

      # Create a location in the container for the Fiesta Application Code
      RUN mkdir /code

      # Make sure that all next commands are run against the /code directory
      WORKDIR /code

      # Copy needed files into the container
      COPY set_privileges.sql /code/set_privileges.sql
      COPY runapp.sh /code

      # Make the runapp.sh executable
      RUN chmod +x /code/runapp.sh

      # Start the application
      ENTRYPOINT [ "/code/runapp.sh"]

      # Expose port 30001 and 3000 to the outside world
      EXPOSE 3001 3000

#. Save the file. Observe that this is the same dockerfile you created for you initial, manual build of the Fiesta container.

#. Following the same process as your **README.md** and **.drone.yml** files, commit the file and push it to the remote **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and observe the new errors.

   .. figure:: images/14.png

#. Return to your **Visual Studio Code (Local)** window.

#. Create the following files missing from the build step:

   - **set_privileges.sql**

      .. code-block:: sql

         grant all privileges on FiestaDB.* to fiesta@'%' identified by 'fiesta';
         grant all privileges on FiestaDB.* to fiesta@localhost identified by 'fiesta';

   - **runapp.sh**

      .. code-block:: bash

         #!/bin/sh

         # Clone the Repo into the container in the /code folder we already created in the dockerfile
         git clone https://github.com/sharonpamela/Fiesta /code/Fiesta

         # Change the configuration from the git clone action
         sed -i 's/REPLACE_DB_NAME/FiestaDB/g' /code/Fiesta/config/config.js
         sed -i "s/REPLACE_DB_HOST_ADDRESS/<IP ADDRESS OF MARIADB SERVER>/g" /code/Fiesta/config/config.js
         sed -i "s/REPLACE_DB_DIALECT/mysql/g" /code/Fiesta/config/config.js
         sed -i "s/REPLACE_DB_USER_NAME/fiesta/g" /code/Fiesta/config/config.js
         sed -i "s/REPLACE_DB_PASSWORD/fiesta/g" /code/Fiesta/config/config.js

         npm install -g nodemon

         # Get ready to start the application
         cd /code/Fiesta
         npm install
         cd /code/Fiesta/client
         npm install

         # Build the app
         npm run build

         # Run the NPM Application
         cd /code/Fiesta
         npm start

   .. note::

      **IMPORTANT!** You need to update *<IP ADDRESS OF MARIADB SERVER>* to the IP address of your **USER**\ *##*\ **-MariaDB_VM** VM in order for your application container to connect to the database.

#. Save both files, commit, and push to the remote **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and observe the image build is successful.

   .. figure:: images/15.png

#. Return to your **Visual Studio Code (Docker VM SSH)** window and open the **Terminal**.

   .. note:: Alternatively, you can SSH to your Docker VM using PuTTY or Terminal.

#. Run ``docker image ls`` to see our image created via the CI/CD pipeline.

   .. figure:: images/16.png

Testing The Image Build
+++++++++++++++++++++++

In a CI/CD pipeline testing is very important and needs to be run automatically. Let's add this step to our **.drone.yml** file. This will ensure that the Docker container can be launched from the image built by **Drone** after each code push.

#. Return to your **Visual Studio Code (Local)** window.

#. Open the **.drone.yml** file.

#. Add the following to the **.drone.yml** file, under the **steps:** section, after the **name: build test image** section.

   .. code-block:: yaml

         - name: Test built container
           image: fiesta_app:${DRONE_COMMIT_SHA:0:6}
           pull: if-not-exists
           environment:
             DB_SERVER: <IP ADDRESS OF MARIADB SERVER>
             DB_PASSWD: fiesta
             DB_USER: fiesta
             DB_TYPE: mysql
           commands:
             - npm version
             - mysql -u$DB_USER -p$DB_PASSWD -h $DB_SERVER FiestaDB -e "select * from Products;"
             - git clone https://github.com/sharonpamela/Fiesta.git /code/Fiesta
             - sed -i 's/REPLACE_DB_NAME/FiestaDB/g' /code/Fiesta/config/config.js
             - sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
             - sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
             - sed -i "s/DB_DOMAIN_NAME/LOCALHOST/g" /code/Fiesta/config/config.js
             - sed -i "s/REPLACE_DB_USER_NAME/$DB_USER/g" /code/Fiesta/config/config.js
             - sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD/g" /code/Fiesta/config/config.js
             - cat /code/Fiesta/config/config.js

   Whitespace in **YAML** files *matters!* When you initially post the content above into the file it may not retain the proper indentation. You can select all or some of the lines and press **Tab** or **Shift-Tab** to indent/unindent multiple lines at once.

   Refer to the image below for a properly indented example.

   .. figure:: images/18.png

   .. note::

      **Visual Code Studio** performs real-time validation of the YAML file. The highlight areas in red represent invalid YAML, indicating the lines need to be indented/unindented.

      .. figure:: images/18b.png

#. Change **<IP ADDRESS OF MARIADB SERVER>** to the IP address of your **USER**\ *##*\ **-MariaDB_VM** VM.

#. Save the file, commit and push to **Gitea**.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and validate the **Test build container** stage completed successfully.

   .. figure:: images/19.png

   Adding this step to **.drone.yml** gets us closer to the goal of delivering *Infrastructure as Code*:

   - We create container using the **fiesta_app** image being automatically built by **Drone**
   - Under **environment**, we define the variables used for the database connection
   - Under **commands**, we define the operations that we are evaluating as part of the test:

Uploading Images To Docker Hub
+++++++++++++++++++++++++++++++

Now that we are programmatically creating and testing our Docker image, the next step is to upload the versioned image to **Docker Hub** so it exists outside of our development environment. Just as Git acts as version control for source code, **Docker Hub** will act as our version control repository for the Docker images themselves.

The following exercise will require you to use your own **Docker Hub** credentials, not the **devnutanix** account referenced in the lab guide screenshots.

Manual Upload
.............

#. Return to your **Visual Studio Code (Docker VM SSH)** window and open the **Terminal**.

   .. note:: Alternatively, you can SSH to your Docker VM using PuTTY or Terminal.

#. Run ``docker login`` and, if prompted, provide *your* **Docker Hub** credentials.

   .. figure:: images/20.png

#. Run ``docker image ls`` to get the list of images on your Docker VM.

   .. figure:: images/21.png

#. Return to **Drone**. The **ACTIVITY FEED** for your latest deployment should still be open, if not, select it and click the **build test image** step.

#. Copy the 6-digit alphanumeric **Tag** from *your* environment, as seen highlighted in the screenshot below.

   .. figure:: images/21-a.png

   .. note::

      Your **Tag** will be different than the one in the screenshot. Every time someone uses the screenshot data in their own lab, and then wonders why their lab doesn't work, a sales rep gets an undeserved raise. Don't let it happen to you.

#. Run ``docker image tag fiesta_app:YOUR-6-DIGIT-TAG YOUR-DOCKERHUB-ACCOUNT-NAME/fiesta_app:1.0``

   This will create a new image which will be tagged with *your* Docker Hub account and **fiesta_app**, as version **1.0**.

#. Run ``docker image ls`` and confirm another instance of your image is listed with the expected **Repository** and **Tag** values.

   .. figure:: images/22.png

#. Run ``docker push YOUR-DOCKERHUB-ACCOUNT-NAME/fiesta_app:1.0`` to initiate to push of the image onto the Dockerhub environment.

#. After the upload completes you should see a confirmation similar to the example below.

   .. figure:: images/23.png

#. In your browser, `sign in to your Docker Hub account <https://hub.docker.com/>`_ and verify your image has been uploaded.

   .. figure:: images/24.png

Now that we can do this manually, let's get **Drone** to do it for us the next time.

CI/CD Upload
************

As we do not want to save our **Docker Hub** credentials in plaintext inside of our **.drone.yml** file, we will use **Drone** to store and retrieve this information dynamically as part of the pipeline.

#. In **Drone**, select your **Fiesta_Application** repo and click the **Settings** tab.

   .. figure:: images/24b.png

#. Under **Secrets**, fill out the following fields (*CASE SENSITIVE!*):

   - **Secret Name** - dockerhub_username
   - **Secret Value** - *Your Docker Hub Username*

   .. figure:: images/24c.png

#. Click **Add A Secret**.

#. Repeat the previous two steps to add (*CASE SENSITIVE!*):

   - **Secret Name** - dockerhub_password
   - **Secret Value** - *Your Docker Hub Password*

   Now that you have added both of these secrets to your **Drone** repo settings, we can add the upload step to our **.drone.yml** file.

#. Return to your **Visual Studio Code (Local)** window and open **.drone.yml**.

#. Add the following to the **.drone.yml** file, under the **steps:** section, after the **name: Test built container** section.

   .. code-block:: yaml

      - name: Push to Dockerhub
        image: docker:latest
        pull: if-not-exists
        environment:
          USERNAME:
            from_secret: dockerhub_username
          PASSWORD:
            from_secret: dockerhub_password
        volumes:
          - name: docker_sock
            path: /var/run/docker.sock
        commands:
          - docker login -u $USERNAME -p $PASSWORD
          - docker image tag fiesta_app:${DRONE_COMMIT_SHA:0:6} $USERNAME/fiesta_app:latest
          - docker image tag fiesta_app:${DRONE_COMMIT_SHA:0:6} $USERNAME/fiesta_app:${DRONE_COMMIT_SHA:0:6}
          - docker push $USERNAME/fiesta_app:${DRONE_COMMIT_SHA:0:6}
          - docker push $USERNAME/fiesta_app:latest

   Again, whitespace in **YAML** files *matters!* Refer to the image below for a properly indented example.

   .. figure:: images/24-1.png

#. Save the file, commit, and push to your **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and validate the **Push to DockerHub** stage completed successfully.

   In **DockerHub**, you should now see 3 **Tags**. **1.0** was your inital, manual push. The **6 digit alphanumeric tag** is your latest CI/CD generated image, and **latest** is a dynamic tag that, logically, corresponds to your latest image upload.

   .. figure:: images/27.png

   .. note::

      If the build fails, it is likely that you have mistyped either the **dockerhub_username** or **dockerhub_password** names or values in **Gitea**. Return to **Drone > nutanix/Fiesta_Application > Settings**, delete the existing secrets, and try again.

After each code push to your repo, you are now using your CI/CD pipeline to build your image, perform basic testing, and push the image to your **Docker Hub** repository. The final stage of the pipeline is to actually deploy the container into production.

Deploying The Container
+++++++++++++++++++++++

This step is very similar to the ``docker run`` command used in :ref:`basic_container` when performing the manual deployment. The key difference is we will also want to stop existing instances of the container to ensure our environment is running the latest, greatest version of the app.

This type of automation is how mature DevOps teams found at organizations like **Netflix** achieve hundreds if not thousands of updates to their production infrastructure every week.

#. Return to your **Visual Studio Code (Local)** window and open **.drone.yml**.

#. Add the following to the **.drone.yml** file, under the **steps:** section, after the **name: Push to Dockerhub** section.

   .. code-block:: yaml

    - name: Deploy newest image
      image: docker:latest
      pull: if-not-exists
      environment:
        USERNAME:
          from_secret: dockerhub_username
      volumes:
        - name: docker_sock
          path: /var/run/docker.sock
      commands:
        - if [ `docker ps | grep Fiesta_App | wc -l` -eq 1 ]; then echo "Stopping existing Docker Container...."; docker stop Fiesta_App; else echo  "Docker container has not been found..."; fi
        - sleep 10
        - docker run --name Fiesta_App --rm -p 5000:3000 -d $USERNAME/fiesta_app:latest

   Again, whitespace in **YAML** files *matters!* Refer to the image below for a properly indented example.

   .. figure:: images/27b.png

#. Save the file, commit, and push to your **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and validate the **Deploy newest image** stage completed successfully.

   .. figure:: images/27c.png

   Adding this step to **.drone.yml** performs the following operations:

   - Search for and stop any running containers named **Fiesta_App**.
   - Print messages via **echo** to provide feedback within **Drone** logs
   - Wait for 10 seconds to allow the Docker engine to remove the existing container
   - Deploy the new container with the following parameters:

      - ``--name`` - Provide the name of the container
      - ``--rm`` - Remove the container after it is stopped
      - ``-p`` - Open external port 5000 and map it to internal port 3000 to provide connectivity to the container
      - ``-d`` - Run the container in the background as a daemon

We now have a complete CI/CD pipeline capable of automatically building, testing, uploading, and deploying our Fiesta application after every code push - *cool!*.

Building With External Variables
+++++++++++++++++++++++++++++++++

Now that you have a functional CI/CD pipeline, we need to consider the parameters that may change during new tests. For instance, the difference between using a pipeline to deploy to your personal development environment versus deploying to a production environment.

Similar to not wanting to store sensitive information like usernames and passwords as part of your repository, environmental variables are often stored externally.

In this exercise you will use the same approach followed to define your **dockerhub_username** and **dockerhub_password** variables to define and store the additional variables required to make your CI/CD build truly dynamic.

The following are parameters being used inside of either **.drone.yml** and/or **runapp.sh**:

   - Docker Hub Username - Already stored in **Drone** as **dockerhub_username**
   - Docker Hub Password - Already stored in **Drone** as **dockerhub_password**
   - Database Server IP
   - Database Name
   - Database Type
   - Database User
   - Database Password

#. In **Drone**, select your **Fiesta_Application** repo and click the **Settings** tab.

#. Under **Secrets**, add the following secrets (*CASE SENSITIVE!*):

   - **db_server_ip** - *Your USER##-MariaDB_VM IP Address*
   - **db_passwd** - fiesta
   - **db_user** - fiesta
   - **db_type** - mysql
   - **db_name** - FiestaDB

   .. figure:: images/28.png

#. Return to your **Visual Studio Code (Local)** window and open **.drone.yml**.

#. Overwrite **ALL** of the contents of the file with the following:

   .. code-block:: yaml

      kind: pipeline
      name: default

      clone:
        skip_verify: true

      steps:

        - name: build test image
          image: docker:latest
          pull: if-not-exists
          volumes:
            - name: docker_sock
              path: /var/run/docker.sock
          commands:
            - docker build -t fiesta_app:${DRONE_COMMIT_SHA:0:6} .

        - name: Test local built container
          image: fiesta_app:${DRONE_COMMIT_SHA:0:6}
          pull: if-not-exists
          environment:
            USERNAME:
              from_secret: dockerhub_username
            PASSWORD:
              from_secret: dockerhub_password
            DB_SERVER:
              from_secret: db_server_ip
            DB_PASSWD:
              from_secret: db_passwd
            DB_USER:
              from_secret: db_user
            DB_TYPE:
              from_secret: db_type
            DB_NAME:
              from_secret: db_name
          commands:
            - npm version
            - mysql -u$DB_PASSWD -p$DB_USER -h $DB_SERVER $DB_NAME -e "select * from Products;"
            - git clone https://github.com/sharonpamela/Fiesta /code/Fiesta
            - if [ `echo $DB_PASSWD | grep "/" | wc -l` -gt 0 ]; then DB_PASSWD=$(echo "${DB_PASSWD//\//\\/}"); fi
            - sed -i 's/REPLACE_DB_NAME/FiestaDB/g' /code/Fiesta/config/config.js
            - sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
            - sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
            - sed -i "s/REPLACE_DB_USER_NAME/$DB_USER/g" /code/Fiesta/config/config.js
            - sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD/g" /code/Fiesta/config/config.js

        - name: Push to Dockerhub
          image: wessenstam/docker:latest
          pull: if-not-exists
          environment:
            USERNAME:
              from_secret: dockerhub_username
            PASSWORD:
              from_secret: dockerhub_password
          volumes:
            - name: docker_sock
              path: /var/run/docker.sock
          commands:
            - docker login -u $USERNAME -p $PASSWORD
            - docker image tag fiesta_app:${DRONE_COMMIT_SHA:0:6} $USERNAME/fiesta_app:latest
            - docker image tag fiesta_app:${DRONE_COMMIT_SHA:0:6} $USERNAME/fiesta_app:${DRONE_COMMIT_SHA:0:6}
            - docker push $USERNAME/fiesta_app:${DRONE_COMMIT_SHA:0:6}
            - docker push $USERNAME/fiesta_app:latest

        - name: Deploy newest image
          image: wessenstam/docker:latest
          pull: if-not-exists
          environment:
            USERNAME:
              from_secret: dockerhub_username
            PASSWORD:
              from_secret: dockerhub_password
            DB_SERVER:
              from_secret: db_server_ip
            DB_PASSWD:
              from_secret: db_passwd
            DB_USER:
              from_secret: db_user
            DB_TYPE:
              from_secret: db_type
            DB_NAME:
              from_secret: db_name
          volumes:
            - name: docker_sock
              path: /var/run/docker.sock
          commands:
            - if [ `docker ps | grep Fiesta_App | wc -l` -eq 1 ]; then echo "Stopping existing Docker Container...."; docker stop Fiesta_App; else echo "Docker container has not been found..."; fi
            - sleep 10
            - docker run --name Fiesta_App --rm -p 5000:3000 -d -e DB_SERVER=$DB_SERVER -e DB_USER=$DB_USER -e DB_TYPE=$DB_TYPE -e DB_PASSWD=$DB_PASSWD -e DB_NAME=$DB_NAME $USERNAME/fiesta_app:latest

      volumes:
      - name: docker_sock
        host:
          path: /var/run/docker.sock

#. Save the file.

   Observe that the **envrionment** section of each **step** maps the **Secret Names** from **Drone** to a variable name that can be referenced within the container image.

#. Open the **runapp.sh** file and overwrite **ALL** of the contents of the file with the following:

   .. code-block:: bash

      #!/bin/sh

      # If there is a "/" in the password or username we need to change it otherwise sed goes haywire
      if [ `echo $DB_PASSWD | grep "/" | wc -l` -gt 0 ]
          then
              DB_PASSWD1=$(echo "${DB_PASSWD//\//\\/}")
          else
              DB_PASSWD1=$DB_PASSWD
      fi

      if [ `echo $DB_USER | grep "/" | wc -l` -gt 0 ]
          then
              DB_USER1=$(echo "${DB_USER//\//\\/}")
          else
              DB_USER1=$DB_USER
      fi

      # Clone the Repo into the container in the /code folder we already created in the dockerfile
      git clone https://github.com/sharonpamela/Fiesta /code/Fiesta

      # Change the Fiesta configuration code so it works in the container
      sed -i "s/REPLACE_DB_NAME/$DB_NAME/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_USER_NAME/$DB_USER1/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD1/g" /code/Fiesta/config/config.js

      # Install the nodemon package
      npm install -g nodemon

      # Get ready to start the application
      cd /code/Fiesta
      npm install
      cd /code/Fiesta/client
      npm install

      # Build the app
      npm run build

      # Run the NPM Application
      cd /code/Fiesta
      npm start

#. Save the file.

   In this script you see the same variables configured in **.drone.yml** being referenced.

#. Commit and push the files to your **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and monitor the deployment takes place using the variables defined in **.drone.yml**.

   .. figure:: images/29.png

#. To monitor the status of your **Fiesta_App** container after being launched by **Drone**, return to your **Visual Studio Code (Docker VM SSH)** window and open the **Terminal**.

   .. note:: Alternatively, you can SSH to your Docker VM using PuTTY or Terminal.

#. From the SSH session, run ``docker logs --follow Fiesta_App``.

   It will take approximately 2-3 minutes for the application to start.

#. Once you see a message similar to the image below, open \http://*<IP ADDRESS DOCKER VM>*:5000/Products and validate you are able to access the Fiesta app.

   .. figure:: images/30.png

.. raw:: html

    <H1><font color="#B0D235"><center>Congratulations!</center></font></H1>

You have now built a complete CI/CD pipeline capable of the following:

- Integrating a rich text editor into the development and deployment workflow :fa:`thumbs-up`
- Building, testing, and deploying the environment after every code push :fa:`thumbs-up`
- Automatically uploading the images to Docker Hub, making it easy to deploy to new development environments :fa:`thumbs-up`
- Providing environment variables from outside of the build environment :fa:`thumbs-up`
- The start of the container takes a long time :fa:`thumbs-down`

In the next exercise, we'll see what can be done to optimize the container start time to make your CI/CD pipeline more efficient!
