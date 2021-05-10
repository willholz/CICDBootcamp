.. _phase4_container:

------------------------
Optimizing the Container
------------------------

While the CI/CD pipeline is now capable of automating the build, test, upload to your Docker Hub registry, and deployment steps, you may have noticed it still takes multiple minutes for the **Fiesta_App** to be ready for use. In environments where you could see thousands of code pushes per day, *minutes matter!*

As you add additional containerized services to the pipeline, where operations are executed begin to have a significant impact on optimizing the build and deployment times of your application.

Currently, the container clones and builds the application source code *after* the container starts. We can shift these operations into the container image build process to decrease the time required for the running container to become ready.

In this exercise you will:

   - Update the **dockerfile** to include the Fiesta installation commands
   - Update **runapp.sh** to remove the Fiesta installation commands
   - Update **.drone.yml** to remove irrelevant image test commands
   - Test your updated build

Updating Fiesta_App Files
+++++++++++++++++++++++++

#. Return to your **Visual Studio Code (Local)** window and open **dockerfile**.

#. Overwrite **ALL** of the contents of the file with the following:

   .. code-block:: yaml

      # This dockerfile multi step is to start the container faster as the runapp.sh doesn't have to run all npm steps

      # Grab the Alpine Linux OS image and name the container base
      FROM wessenstam/ntnx-alpine:latest as base

      # Install needed packages
      RUN apk add --no-cache --update nodejs npm git

      # Create and set the working directory
      RUN mkdir /code
      WORKDIR /code

      # Get the Fiesta Application in the container
      RUN git clone https://github.com/sharonpamela/Fiesta.git /code/Fiesta

      # Get ready to install and build the application
      RUN cd /code/Fiesta && npm install
      RUN cd /code/Fiesta/client && npm install
      RUN cd /code/Fiesta/client && npm audit fix
      RUN cd /code/Fiesta/client && npm fund
      RUN cd /code/Fiesta/client && npm update
      RUN cd /code/Fiesta/client && npm run build

      # Grab the Alpine Linux OS image and name it Final_Image
      FROM wessenstam/ntnx-alpine:latest as Final_Image

      # Install some needed packages
      RUN apk add --no-cache --update nodejs npm mysql-client

      # Get the NMP nodemon and install it
      RUN npm install -g nodemon

      # Copy the earlier created application from the first step into the new container
      COPY --from=base /code /code

      # Copy the starting app
      COPY runapp.sh /code
      RUN chmod +x /code/runapp.sh
      WORKDIR /code

      # Start the application
      ENTRYPOINT [ "/code/runapp.sh"]
      EXPOSE 3001 3000

#. Save the file.

   .. note::

      We will **NOT** push the changes until all files have been updated.

   Now we see that the Fiesta application source code will be cloned and built on the Docker VM and *then* copied into the container on the ``COPY --from=base /code /code`` line.

   Not only will this decrease the start time of the application, it will also decrease the total size. This is because many additional *temporary* packages are downloaded by **npm** as part of the build process which are not automatically removed after the build has completed.

#. Open **runapp.sh** and overwrite **ALL** of the contents of the file with the following:

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

      # Change the Fiesta configuration code so it works in the container
      sed -i "s/REPLACE_DB_NAME/$DB_NAME/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_USER_NAME/$DB_USER1/g" /code/Fiesta/config/config.js
      sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD1/g" /code/Fiesta/config/config.js

      # Run the NPM Application
      cd /code/Fiesta
      npm start

#. Save the file.

   The only thing the start-up script for our container is now responsible for is updating the **config.js** file with the environment variables and starting the application.

#. Open **.drone.yml**.

#. Under **steps > name: Test local built container > commands**, remove the line ``- git clone https://github.com/sharonpamela/Fiesta /code/Fiesta``.

   .. figure:: images/5.png

   This test is no longer needed as the source code as is now being cloned from GitHub outside of the container image.

#. Save the file.

Testing The Optimizations
+++++++++++++++++++++++++

#. Commit and push your 3 updated files to your **Gitea** repo.

#. In **Drone > nutanix/Fiesta_Application > ACTIVITY FEED**, note the the **build test image** stage now takes significantly longer as this is where we have shifted a majority of the operations.

   .. figure:: images/1.png

   This is a reasonable trade-off as for every build in an environment, you will likely have multiple deployments (development environments, user acceptance testing, production, etc.).

#. After the **Deploy newest image** stage is complete, return to your **Visual Studio Code (Docker VM SSH)** window and open the **Terminal**.

   .. note:: Alternatively, you can SSH to your Docker VM using PuTTY or Terminal.

#. Run ``docker image ls`` to list the images.

   .. figure:: images/3.png

   In the example above, the size of the image decreased by nearly 100MB. Again this is due to eliminating all of the additional temporary packages downloaded by **npm** when performing the application build inside of the container.

   Next we'll test how quickly the new image is able to start the Fiesta app.

#. Run ``docker stop Fiesta_App`` to stop and remove your container.

#. You can run ``docker ps --all`` to validate **Fiesta_App** container is no longer present.

   You should expect to see only your **drone**, **drone-runner-docker**, **gitea**, and **mysql** containers.

#. Copy and paste the script below into a temporary text file and update the **DB_SERVER** and **USERNAME** variables to match your environment and **Docker Hub** account.

   .. code-block:: bash

      DB_SERVER=<IP ADDRESS OF MARIADB VM>
      DB_NAME=FiestaDB
      DB_USER=fiesta
      DB_PASSWD=fiesta
      DB_TYPE=mysql
      USERNAME=<DOCKERHUB USERNAME>
      docker run --name Fiesta_App --rm -p 5000:3000 -d -e DB_SERVER=$DB_SERVER -e DB_USER=$DB_USER -e DB_TYPE=$DB_TYPE -e DB_PASSWD=$DB_PASSWD -e DB_NAME=$DB_NAME $USERNAME/fiesta_app:latest && docker logs --follow Fiesta_App

#. Paste the updated script into your SSH terminal session and press **Return** to execute the final command.

   The app should start in ~15 seconds, as indicated by ``You can now view client in the browser`` output from your terminal session. *That's significantly faster than the 3+ minutes it took previously!*

#. Optionally, if you want to compare the start time of your previous build:

   - Press **CTRL+C** to stop the ``docker log`` command
   - Run ``docker stop Fiesta_App``
   - Run ``docker image ls`` and note the **TAG** of one of your previous versions of the image, as indicated by its larger file size

      .. figure:: images/6.png

   - In the following command, replace **LATEST** with the **TAG** value from the previous step run ``docker run --name Fiesta_App --rm -p 5000:3000 -d -e DB_SERVER=$DB_SERVER -e DB_USER=$DB_USER -e DB_TYPE=$DB_TYPE -e DB_PASSWD=$DB_PASSWD -e DB_NAME=$DB_NAME $USERNAME/fiesta_app:LATEST && docker logs --follow Fiesta_App``

   - Run the command

   This version should take *much* longer than the optimized container image.

.. raw:: html

    <H1><font color="#B0D235"><center>Congratulations!</center></font></H1>

You've addressed the final issue in our CI/CD pipeline by optimizing the time it takes to deploy the application from the Docker container. :fa:`thumbs-up` What now?

Up to this point in the lab, every build has been dependent on the pre-deployed "production" version of our MariaDB database. In the next exercise, we'll take advantage of **Nutanix Era** to provide database cloning as part of the pipeline.
