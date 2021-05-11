.. _phase5_era:

---------------------------------------
Creating Development Databases with Era
---------------------------------------

While any developer in the organization could now spin up their own development environments using the CI/CD pipeline you've built, having them all point to the same "production" database is surely a recipe for disaster.

In this exercise we'll leverage **Nutanix Era** to programmatically generate clones of our source database for use within a development environment. To accomplish this we will need to:

   - Use **Era** to determine the API calls to clone the source MariaDB database
   - Define additional variables in **Drone** for our **Era** environment
   - Create a branch of our **Fiesta_APP** repo to define developer environment versions of **runapp.sh** and **dockerfile**
   - Update **.drone.yml** to include the additional step to provision the developer container, which will include the **Era** cloning operation

Understanding The Era Clone API Call
++++++++++++++++++++++++++++++++++++

Before determining the API call required to perform the clone operation, you will create a manual snapshot of your database's **Time Machine** to clone.

In a production environment you would likely want to clone a database VM using the latest **Point In Time** based on the continuous data protection provided by Era. Doing this would require additional API calls to return the latest available timestamp for the clone. This was eliminated to shorten the lab.

#. Refer to :ref:`clusterdetails` for your **Era** IP and **admin** credentials.

#. Open **Era** in your browser and log in as **admin**.

#. Click **Dashboard** in the toolbar and select **Time Machines** from the dropdown menu.

   .. figure:: images/0.png

#. Select your **User**\ *##*\ **-FiestaDB_TM**.

#. Click the **Actions** dropdown menu and select **Snapshot**.

#. Specify **First-Snapshot** as the **Snapshot Name**.

   .. figure:: images/2a.png

   .. note::

      **IMPORTANT!** It is important to use this exact snapshot name, as it is referenced later in the exercise.

#. Click **Create**.

   It should take ~1 minute for the snapshot creation operation to complete. You can monitor this in **Operations**, which can be accessed via the **Era** dropdown menu in the toolbar.

   .. figure:: images/2b.png

#. When the **Create Snapshot** operation completes, return to **Time Machines**.

#. Select your **User**\ *##*\ **-FiestaDB_TM** and click **Actions > Create a Clone of MariaDB Instance**.

   .. figure:: images/3.png

   After we step through the GUI to perform the cloning operation, **Era** will provide us with the equivalent code required to perform the operation via API.

#. Under **Time/Snapshot**, select **Snapshot** and **First-Snapshot** from the dropdown menu.

   .. figure:: images/3b.png

#. Click **Next**.

#. Under **Database Server VM**, fill out the following fields:

   - **Database Server VM** - Create New Server
   - **Database Server VM Name** - *Leave Default*
   - **Compute Profile** - CUSTOM_EXTRA_SMALL
   - **Network Profile** - Era_Managed_MariaDB
   - Select **Provide SSH Public Key Through Text** and paste the following:

     .. code-block:: SSH

        ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmhJS2RbHN0+Cz0ebCmpxBCT531ogxhxv8wHB+7Z1G0I77VnXfU+AA3x7u4gnjbZLeswrAyXk8Rn/wRMyJNAd7FTqrlJ0Imd4puWuE2c+pIlU8Bt8e6VSz2Pw6saBaECGc7BDDo0hPEeHbf0y0FEnY0eaG9MmWR+5SqlkepgRRKN8/ipHbi5AzsQudjZg29xra/NC/BHLAW/C+F0tE6/ghgtBKpRoj20x+7JlA/DJ/Ec3gU0AyYcvNWlhlR+qc83lXppeC1ie3eb9IDTVbCI/4dXHjdSbhTCRu0IwFIxPGK02BL5xOVTmxQyvCEOn5MSPI41YjJctUikFkMgOv2mlV root@centos

   .. figure:: images/3c.png

#. Click **Next**.

#. Under **Database Server VM**, fill out the following fields:

   - **Name** - *Leave Default*
   - **New ROOT Password** - nutanix/4u
   - **Database Parameter Profile** - DEFAULT_MARIADB_PARAMS

   .. raw:: html

      <br><strong><font color="red">DO NOT PRESS CLONE!</font></strong><br><br>

   .. figure:: images/3d.png

#. Click **API Equivalent**.

   .. figure:: images/4.png

   The **JSON Data** on the left shows the payload of the REST API call on the right. Within the payload you can observe a number of values that will need to be provided as part of the final API call to clone the database, such as **timeMachineId** and **snapshotId**.

   Additionally, we'll need to provide the details of our **Era** environment to the build as additional **Drone** variables.

#. Click the **Close** button and the **X** to close the **Create Clone of MariaDB Instance from Time Machine** window.

   .. raw:: html

      <br><strong><font color="red">DO NOT PRESS CLONE!</font></strong><br><br>

   **Era** makes it very simple to understand how to perform operation programmatically by providing the API equivalent of the selections you've made in the UI. We'll use a variation of this API call within our CI/CD development build.

Adding Drone Secrets
++++++++++++++++++++

#. Refer to :ref:`clusterdetails` for your **Era** IP and **admin** credentials.

#. In **Drone**, select your **Fiesta_Application** repo and click the **Settings** tab.

#. Under **Secrets**, add the following secrets (*CASE SENSITIVE!*):

   - **era_ip** - *Your Era IP Address*
   - **era_user** - admin
   - **era_password** - *Your Era admin Password*
   - **initials** - *Your User## prefix of your User##-FiestaDB_TM in Era* (ex. User01)

   .. figure:: images/17.png

   .. raw:: html

      <br><strong><font color="red">Do NOT use your initials. The value needs to be the User## prefix found in Era as the API calls later in the exercise will be searching for your exact User##-FiestaDB_TM to clone.</font></strong><br><br>

   You should now have 11 **Secrets** in total.

Adding The Dev Container Deployment
+++++++++++++++++++++++++++++++++++

We will now update our **.drone.yml** with an additional **step** to conditionally deploy a development environment, which will include the database clone.

#. Return to your **Visual Studio Code (Local)** window in your **USER**\ *##*\ **-WinToolsVM** and open **.drone.yml**.

#. Copy and paste below content over the exiting content in the **.drone.yml** file:

   .. code-block:: yaml

    kind: pipeline
    name: default

    clone:
      skip_verify: true

    steps:

      - name: Build Image (Prod)
        image: ntnxgteworkshops/docker:latest

        pull: if-not-exists
        volumes:
          - name: docker_sock
            path: /var/run/docker.sock
        commands:
          - docker build -t fiesta_app:${DRONE_COMMIT_SHA:0:6} .
        when:
          branch:
            - master

      - name: Build Image (Dev)
        image: ntnxgteworkshops/docker:latest

        pull: if-not-exists
        volumes:
          - name: docker_sock
            path: /var/run/docker.sock
        commands:
          - docker build -t fiesta_app_dev:${DRONE_COMMIT_SHA:0:6} -f dockerfile-dev .
        when:
          branch:
            - dev

      - name: Test container (Prod)
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
          - if [ `echo $DB_PASSWD | grep "/" | wc -l` -gt 0 ]; then DB_PASSWD=$(echo "${DB_PASSWD//\//\\/}"); fi
          - sed -i 's/REPLACE_DB_NAME/FiestaDB/g' /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_USER_NAME/$DB_USER/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD/g" /code/Fiesta/config/config.js
        when:
          branch:
            - master

      - name: Test container (Dev)
        image: fiesta_app_dev:${DRONE_COMMIT_SHA:0:6}
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
          - if [ `echo $DB_PASSWD | grep "/" | wc -l` -gt 0 ]; then DB_PASSWD=$(echo "${DB_PASSWD//\//\\/}"); fi
          - sed -i 's/REPLACE_DB_NAME/FiestaDB/g' /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_HOST_ADDRESS/$DB_SERVER/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_DIALECT/$DB_TYPE/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_USER_NAME/$DB_USER/g" /code/Fiesta/config/config.js
          - sed -i "s/REPLACE_DB_PASSWORD/$DB_PASSWD/g" /code/Fiesta/config/config.js
        when:
          branch:
            - dev

      - name: Push to Dockerhub (Prod)
        image: ntnxgteworkshops/docker:latest

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
        when:
          branch:
            - master

      - name: Deploy Prod image
        image: ntnxgteworkshops/docker:latest
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
          - if [ `docker ps | grep Fiesta_App | wc -l` -eq 1 ]; then echo "Stopping existing Docker Container...."; docker stop Fiesta_App; sleep 10; else echo "Docker container has not been found..."; fi
          -
          - docker run --name Fiesta_App --rm -p 5000:3000 -d -e DB_SERVER=$DB_SERVER -e DB_USER=$DB_USER -e DB_TYPE=$DB_TYPE -e DB_PASSWD=$DB_PASSWD -e DB_NAME=$DB_NAME $USERNAME/fiesta_app:latest
        when:
          branch:
            - master

      - name: Deploy Dev image
        image: ntnxgteworkshops/docker:latest
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
          ERA_IP:
            from_secret: era_ip
          ERA_USER:
            from_secret: era_user
          ERA_PASSWORD:
            from_secret: era_password
          INITIALS:
            from_secret: initials
        volumes:
          - name: docker_sock
            path: /var/run/docker.sock
        commands:
          - if [ `docker ps | grep fiesta_app_dev | wc -l` -eq 1 ]; then echo "Stopping existing Docker Container...."; docker stop fiesta_app_dev; sleep 10; else echo "Docker container has not been found..."; fi
          - docker run -d -v /tmp:/tmp --rm --name fiesta_app_dev -p 5050:3000 -e DB_SERVER=$DB_SERVER -e DB_USER=$DB_USER -e DB_TYPE=$DB_TYPE -e DB_PASSWD=$DB_PASSWD -e DB_NAME=$DB_NAME -e initials=$INITIALS -e era_ip=$ERA_IP -e era_admin=$ERA_USER -e era_password=$ERA_PASSWORD fiesta_app_dev:${DRONE_COMMIT_SHA:0:6}
        when:
          branch:
            - dev

    volumes:
    - name: docker_sock
      host:
        path: /var/run/docker.sock


   The new **.drone.yml** adds two key changes:

   - Steps are now run conditionally when the **branch** of the Git push is **Master** or **dev**. Up to this point, all commits have been to the **master** branch.

   - If branch is **dev**, the following changes in the steps, compared to earlier runs, are:

     - Change the name of the build image to **fiesta_app_dev**
     - Use a different **dockerfile** to build the image (**dockerfile-dev**)
     - Don't push the image to your **Docker Hub** registry
     - Start a container using the dev built container on port **5050**, not **5000**.
     - Name the container **fiesta_app_dev**

#. Save the file. Commit and push to your **Gitea** repo.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** and observe that the steps completed are for the **Prod** environment.

   .. figure:: images/7.png

Creating A New Branch
+++++++++++++++++++++

Now that we know our CI/CD pipeline can conditionally perform different actions based on branch, we will create a new branch within the repo to define the development build. This will allow us to deploy an alternate **runapp.sh** script to deploy and use the MariaDB clone.

..   As we are mimicking the full development of the applicaiton, we are going to create a new branch. This branch will be used to do a few things:

   - Change the creation of the development container
   - Run a different start script which will:

     - Deploy a clone of the MariaDB server, if there is none
     - Use the cloned MariaDB server and not the MariaDB production server for the development of our application

     - Don't upload the container onto our DockerHub repo as it has no Production value

#. Return to your **Visual Studio Code (Local)** window in your **USER**\ *##*\ **-WinToolsVM**.

#. Close any open files in **Visual Studio Code (Local)**.

#. In the bottom, left-hand corner of **Visual Studio Code**, click **master**.

   .. figure:: images/8b.png

#. Select **+ Create new branch...**

#. Specify **dev** as the **Branch Name** and press **Return** to create the branch.

   .. figure:: images/8c.png

   Note in the bottom, left-hand corner the branch has changed to **dev**. In the **Explorer** you will have all the same files as the **master** branch, but we can make independent changes to the repo.

   .. figure:: images/8d.png

Creating Development runapp Script
++++++++++++++++++++++++++++++++++

As seen in Era, there are multiple variables that need to be populated in order to successfully execute the clone operation. To simplify the lab, these steps have been provided for you.

#. Create a new file named **runapp-dev.sh** in the **Fiesta_Application** directory.

#. Copy and paste the contents below into the file:

   .. code-block:: bash

      #!/bin/sh

      # Install curl and jq package as we need it
      apk add curl jq

      # Function area
      function waitloop {
        op_answer="$1"
        loop=$2
        # Get the op_id from the task
        op_id=$(echo $op_answer | jq '.operationId' | tr -d \")


        # Checking on error. if we have received an error, show it and exit 1
        if [[ -z $op_id ]]
        then
            echo "We have received an error message. The reply from the Era system has been "$op_answer" .."
            exit 1
        else
          counter=1
          # Checking routine to see that the registration in Era worked
          while [[ $counter -le $loop ]]
          do
              ops_status=$(curl -k --silent https://${era_ip}/era/v0.9/operations/${op_id} -H 'Content-Type: application/json'  --user $era_admin:$era_password | jq '.["percentageComplete"]' | tr -d \")
              if [[ $ops_status == "100" ]]
              then
                  ops_status=$(curl -k --silent https://${era_ip}/era/v0.9/operations/${op_id} -H 'Content-Type: application/json'  --user $era_admin:$era_password | jq '.status' | tr -d \")
                  if [[ $ops_status == "5" ]]
                  then
                     echo "Database and Database server have been registreed in Era..."
                     break
                  else
                     echo "Database and Database server registration not correct. Please look at the Era GUI to find the reason..."
                     exit 1
                  fi
              else
                  echo "Operation still in progress, it is at $ops_status %... Sleep for 30 seconds before retrying.. ($counter/$loop)"
                  sleep 30
              fi
              counter=$((counter+1))
          done
          if [[ $counter -ge $loop ]]
          then
            echo "We have tried for "$(expr $loop / 2)" minutes to register the MariaDB server and Database, but were not successful. Please look at the Era GUI to see if anything has happened..."
          fi
      fi
      }

      # Variables received from the environmental values via the Drone Secrets
      # era_ip, era_user, era_password and initials

      # Create VM-Name
      vm_name_dev=$initials"-MariaDB_DEV-VM"
      db_name_prod=$initials"-FiestaDB"
      db_name_dev=$initials"-FiestaDB_DEV"


      # Get the UUID of the Era server
      era_uuid=$(curl -k --insecure --silent https://${era_ip}/era/v0.9/clusters -H 'Content-Type: application/json' --user $era_admin:$era_password | jq -r '.[] | select(.name=="EraCluster")| .id')

      # Get the UUID of the network called Era_Managed_MariaDB
      network_id=$(curl --silent -k "https://${era_ip}/era/v0.9/profiles?type=Network&name=Era_Managed_MariaDB" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq '.id' | tr -d \")

      # Get the UUID for the ComputeProfile
      compute_id=$(curl --silent -k "https://${era_ip}/era/v0.9/profiles?&type=Compute&name=CUSTOM_EXTRA_SMALL" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq '.id' | tr -d \")

      # Get the UUID for the DatabaseParameter ID
      db_param_id=$(curl --silent -k "https://${era_ip}/era/v0.9/profiles?engine=mariadb_database&name=DEFAULT_MARIADB_PARAMS" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq '.id' | tr -d \")

      # Get the UUID of the timemachine
      db_name_tm=$initials"-FiestaDB_TM"
      tms_id=$(curl --silent -k "https://${era_ip}/era/v0.9/tms" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq --arg db_name_tm $db_name_tm '.[] | select (.name==$db_name_tm) .id' | tr -d \")

      # Get the UUID of the First-Snapshot for the TMS we just found
      snap_id=$(curl --silent -k "https://${era_ip}/era/v0.9/snapshots" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq --arg tms_id $tms_id '.[] | select (.timeMachineId==$tms_id) | select (.name=="First-Snapshot") .id' | tr -d \")

      # Now that we have all the needed parameters we can check if there is a clone named INITIALS-FiestaDB_DEV
      clone_id=$(curl --silent -k "https://${era_ip}/era/v0.9/clones" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq --arg db_name_dev $db_name_dev '.[] | select (.name==$db_name_dev) .id' | tr -d \")

      # Getting the parameters outside of the container
      echo "------------------------------------" >> /tmp/test.txt
      echo "Era IP :"$era_ip  >> /tmp/test.txt
      echo "Era Username :"$era_admin >> /tmp/test.txt
      echo "Era_password :"$era_password >> /tmp/test.txt
      echo "Era UUID :"$era_uuid >> /tmp/test.txt
      echo "Network ID :"$network_id >> /tmp/test.txt
      echo "Compute ID :"$compute_id >> /tmp/test.txt
      echo "DB Parameters :"$db_name_tm >> /tmp/test.txt
      echo "TMS ID :"$tms_id >> /tmp/test.txt
      echo "Snap ID :"$snap_id >> /tmp/test.txt
      echo "Clone ID :"$clone_id >> /tmp/test.txt
      echo "Initials :"$initials >> /tmp/test.txt
      echo "------------------------------------" >> /tmp/test.txt

      # Check if there is a clone already. if not, start the clone process
      if [[ -z $clone_id ]]
      then
          # Clone call of the MariaDB
          opanswer=$(curl --silent -k -X POST \
              "https://${era_ip}/era/v0.9/tms/$tms_id/clones" \
              -H 'Content-Type: application/json' \
              --user $era_admin:$era_password  \
              -d \
              '{"name":"'$db_name_dev'","description":"Dev clone from the '$db_name_prod'","createDbserver":true,"clustered":false,"nxClusterId":"'$era_uuid'","sshPublicKey":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmhJS2RbHN0+Cz0ebCmpxBCT531ogxhxv8wHB+7Z1G0I77VnXfU+AA3x7u4gnjbZLeswrAyXk8Rn/wRMyJNAd7FTqrlJ0Imd4puWuE2c+pIlU8Bt8e6VSz2Pw6saBaECGc7BDDo0hPEeHbf0y0FEnY0eaG9MmWR+5SqlkepgRRKN8/ipHbi5AzsQudjZg29xra/NC/BHLAW/C+F0tE6/ghgtBKpRoj20x+7JlA/DJ/Ec3gU0AyYcvNWlhlR+qc83lXppeC1ie3eb9IDTVbCI/4dXHjdSbhTCRu0IwFIxPGK02BL5xOVTmxQyvCEOn5MSPI41YjJctUikFkMgOv2mlV root@centos","dbserverId":null,"dbserverClusterId":null, "dbserverLogicalClusterId":null,"timeMachineId":"'$tms_id'","snapshotId":"'$snap_id'",  "userPitrTimestamp":null,"timeZone":"Europe/Amsterdam","latestSnapshot":false,"nodeCount":1,"nodes":[{"vmName":"'$vm_name_dev'",  "computeProfileId":"'$compute_id'","networkProfileId":"'$network_id'","newDbServerTimeZone":null,   "nxClusterId":"'$era_uuid'","properties":[]}],"actionArguments":[{"name":"vm_name","value":"'$vm_name_dev'"}, {"name":"dbserver_description","value":"Dev clone from the '$vm_name'"},{"name":"db_password","value":"nutanix/4u"}],"tags":[],"newDbServerTimeZone":"UTC","computeProfileId":"'$compute_id'","networkProfileId":"'$network_id'",    "databaseParameterProfileId":"'$db_param_id'"}')

          # Call the waitloop function
          waitloop "$opanswer" 30
      fi

      # Let's get the IP address of the cloned database server
      cloned_vm_ip=$(curl --silent -k "https://${era_ip}/era/v0.9/dbservers" -H 'Content-Type: application/json' --user $era_admin:$era_password | jq --arg clone_name $vm_name_dev '.[] | select (.name==$clone_name) .ipAddresses[0]' | tr -d \")

      # Getting the parameters outside of the container
      echo "Era IP :"$era_ip  >> /tmp/test.txt
      echo "Era Username :"$era_admin >> /tmp/test.txt
      echo "Era_password :"$era_password >> /tmp/test.txt
      echo "Era UUID :"$era_uuid >> /tmp/test.txt
      echo "Network ID :"$network_id >> /tmp/test.txt
      echo "Compute ID :"$compute_id >> /tmp/test.txt
      echo "DB Parameters :"$db_name_tm >> /tmp/test.txt
      echo "TMS ID :"$tms_id >> /tmp/test.txt
      echo "Snap ID :"$snap_id >> /tmp/test.txt
      echo "Clone ID :"$clone_id >> /tmp/test.txt
      echo "Initials :"$initials >> /tmp/test.txt

      DB_SERVER=$cloned_vm_ip
      echo "Cloned DB server ip: "$DB_SERVER >> /tmp/test.txt

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

   This script will:

   - Check if there is already a clone of **User**\ *##*\ **-MariaDB_VM** deployed, otherwise create one with the following naming scheme:

    - **User**\ *##*\ **-MariaDB_DEV-VM** as the provisioned Database Server
    - **User**\ *##*\ **-FiestaDB_DEV** as the name of the cloned Database
    - **User**\ *##*\ **-FiestaDB_DEV_TM** as the name of the Time Machine of the cloned Database

   - Set **config.js** for Fiesta to use the cloned database as its database server
   - Start the Fiesta application

#. Save the file.

   .. raw:: html

      <br><strong><font color="red">DO NOT COMMIT/PUSH YET!</font></strong><br><br>

Creating Development Dockerfile
+++++++++++++++++++++++++++++++

Now we need to make sure that the development container is using the newly created **runapp-dev.sh** file.

#. Create a new file named **dockerfile-dev** in the **Fiesta_Application** directory.

#. Copy and paste the contents below into the file:

   .. code-block:: docker

      # This dockerfile multi step is to start the container faster as the runapp.sh doesn't have to run all npm steps

      # Grab the Alpine Linux OS image and name the container base
      FROM ntnxgteworkshops/alpine:latest as base

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
      RUN cd /code/Fiesta/client && npm run build

      # Grab the Alpine Linux OS image and name it Final_Image
      FROM ntnxgteworkshops/alpine:latest as Final_Image

      # Install some needed packages
      RUN apk add --no-cache --update nodejs npm mysql-client

      # Get the NMP nodemon and install it
      RUN npm install -g nodemon

      # Copy the earlier created application from the first step into the new container
      COPY --from=base /code /code

      # Copy the starting app, but dev version
      COPY runapp-dev.sh /code/runapp.sh
      RUN chmod +x /code/runapp.sh
      WORKDIR /code

      # Start the application
      ENTRYPOINT [ "/code/runapp.sh"]
      EXPOSE 3001 3000

   This is nearly identical to your production **dockerfile**. You can see the difference on the ``COPY runapp-dev.sh /code/runapp.sh`` line where **runapp-dev.sh** is copied into the container image as **runapp.sh**.

#. Save the file.

Testing Your Development Build
++++++++++++++++++++++++++++++

#. In **Visual Studio Code (Local)**, commit and push your **runapp-dev.sh** and **dockerfile-dev** files to the **Gitea** repo.

#. When prompted, click **OK** to publish the **dev** branch upstream.

   .. figure:: images/12.png

   This appears because the **dev** branch does not yet exist within your repo in **Gitea**.

#. Return to **Drone > nutanix/Fiesta_Application > ACTIVITY FEED** to monitor the container deployment status.

   .. figure:: images/13.png

#. Once **Deploy Dev image** completes, return to your **Visual Studio Code (Docker VM SSH)** window and open the **Terminal**.

   .. note:: Alternatively, you can SSH to your Docker VM using PuTTY or Terminal.

#. Run ``docker logs --follow fiesta_app_dev``.

   .. figure:: images/14.png

   You should expect to see **Operation still in progress...** as the cloning operation is taking place.

#. Open **Era > Operations** and you should expect to see a **Clone Database** operation for your new **User**\ *##*\ **-FiestaDB_DEV** database.

   .. figure:: images/18.png

#. Once the clone operation is completed, verify in your SSH terminal session that the application has started.

   .. figure:: images/19.png

#. Open \http://*<IP ADDRESS DOCKERVM>*:5050 in your browser to access the development build of your Fiesta application.

   .. figure:: images/20.png

#. Select **Products** and then click the **Add New Product** button.

#. Fill out the following fields to add a new product to the Fiesta database:

   - **Product Name** - The Best Balloons
   - **Suggested Retail Price** - 10000
   - **Product Image URL** - \https://images-na.ssl-images-amazon.com/images/I/61Igt9PNzKL._AC_SL1500_.jpg
   - **Product Comments** - Everybody Knows

#. Click **Submit**.

   You should now see your new product at the bottom of your list of products.

#. In your browser, change the URL to the production application by changing the port number from **5050** to **5000**.

   As expected, the data added to your development database does not appear in production - *nice!*.


.. let's roll the Development database back to the time we created the snapshot.

    Refresh the development database
    --------------------------------

    #. Open your Era instance
    #. Goto **Databases (drop down menu) -> Clones**
    #. Click the radio button in from of your *Initials* **-FiestaDB_DEV** clone
    #. Click the **Refresh** button
    #. Select under **Snapshot** your **First-Snapshot**

       .. figure:: images/16.png

    #. Click **Refresh**
    #. Click **Operations** to follow the process (approx. 5-7 minutes)

.. raw:: html

   <H1><font color="#B0D235"><center>Congratulations!</center></font></H1>

By leveraging **Nutanix Era** as part of your CI/CD pipeline, you are now able to easily deploy clones of your production application database to deliver complete application development environments to your users.

Era could be further exploited as part of the pipeline to provide tasks like automated database refreshes to ensure development clones are using the latest data.

Additionally, you could incorporate Era's multi-cluster management capabilities to provide development environments across multiple sites, including the public cloud with Nutanix Clusters on AWS. The sky is literally the limit _ *get it, get it?!*
