# Bonita Exercice

## User story 1

### a (folder v1)

To launch bonita community edition using K8s, we can create a Deployment file  with it's config specifying the docker Image "Bonita"

Following the best practices , I chose to put ENV variables into a configmap , allowing for separating the pod from it's ENV.

A final service was added to allow traffic to bonita pod .

### b (folder v2)

Now adding Postgres as the database service for Bonita , we need to :

* add a Postgres Statefulset , since statefulset are the best Practice for any stateful application
* add a service to access the postgres pod.

then , going back to the bonita-deployment , it should be updated, by adding the  following env variables:

* DB_HOST :  the postgres service
* DB_PORT : 5432
* DB_NAME ....

by Re-checking the postgres pod , we could see that see that our configuration was valid.

## User story 2

First , I've had to use kubectl exec -it nameOfPod /bin/bash  , to explore the pod's file system and especially check the /opt directory , as there is where the bonita config is stored .

then I've added a configMap 1.bonita-initScript-configmap.yml, this configMap contains the script we are going to use to automate the process of changing verbosity of the loggers

```yaml
  script.sh: |
    #!/bin/bash
    # Path to the log4j2-loggers.xml and log4j2-appenders.xml files
    echo "Im working"
    LOGGERS_XML="/opt/bonita/conf/logs/log4j2-loggers.xml"
    APPENDERS_XML="/opt/bonita/conf/logs/log4j2-appenders.xml"

    # Update log4j2-loggers.xml
    sed -i 's/level="WARN"/level="INFO"/' "$LOGGERS_XML"

    # Update log4j2-appenders.xml
    sed -i 's/level="WARN"/level="INFO"/' "$APPENDERS_XML"

    echo "Log verbosity updated successfully."

```

Now that are we have our script , I've tried to just mount  the configmap as a volume in the pod in the /opt/custom.init.d/ directory and use it from there , but I was encountred with an error specifying that the file system is READ-ONLY, and that a ```chown```  command fails. 

I've started experimenting with solutions online , and I've found that we can add an initContainers to our deployment , specifying required steps to prepare the launch of our bonita pods. in there , I've used a small image "busybox", to copy the script.sh from the configmap volume , and put it into a new emptydir{} volume , the latest will be then mounted into /opt/custom.init.d/ .

by checking the logs , I've found that the script was launched and ran successfully .
