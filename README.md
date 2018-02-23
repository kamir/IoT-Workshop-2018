# IoT-Workshop-2018

The goal of the workshop is:

Enable a data flow from an IoT Edgedevice via Kapua and Kafka to CDH.
Sensordata should be queried via Impala in HUE. This also allows the 
integration of an BI tool using th Impala JDBC connectore.

# Short Version: less than 2 hours 

## Step 0 : Create your virtual machines and Cloudera cluster (in CloudCat)

  1. Open this page in a browser: https://master-01.jenkins.cloudera.com/job/Cluster-Setup/build?delay=0sec (Note that you will need to be connected to the VPN, and will need to authenticate using your Okta credentials).
  2. Under "Project Cluster-Setup", look for the parameter "CLUSTER_SHORTNAME", and enter a string in the textbox which will serve as the prefix for the CloudCat hosts Jenkins will provision on your behalf. You should provision a 3 node cluster by providing a string formatted similar to (but not exactly...choose something unique or your hostname will collide with other people) "jcooperellis-iot-cluster-{1..3}". If you do not modify any other settings, this will provision hosts \[jcooperellis-iot-cluster-1.gce.cloudera.com, jcooperellis-iot-cluster-2.gce.cloudera.com, jcooperellis-iot-cluster-3.gce.cloudera.com\] and install a Cloudera cluster on them. CM & master roles will be installed on jcooperellis-iot-cluster-1.gce.cloudera.com.
  3. Scroll down and look for the parameter "DB". Change the value from default "maria_or_msql" to "postgresql". If you do not do this, the IoT Hub installed in a later step will collide with the CM database and you will not be able to complete the workshop.
  4. Scroll to the bottom of the page and click the button labeled "Build". This will cause the Jenkins job to start. It will first provision your hosts. If you provided a unique hostname you will receive an email within a few minutes confirming that your CloudCat instances have been successfully provisioned. While connected to the VPN, you will then be able to ssh into those instances using credentials root/cloudera"...e.g. byt opening a terminal and typing `ssh root@jcooperellis-iot-cluster-1.gce.cloudera.com`, then typing `cloudera` when prompted for a password.
  5. While your instances should provision within a few minutes, it will likely take a half hour or more for the Cloudera cluster to be installed on those instances, so be patient for that. In the meantime, it is OK to install Kapua and Kura on on your "-3" instance...e.g. "jcooperellis-iot-cluster-3.gce.cloudera.com". Following the instructions in step 4, ssh into your "-3" instance and continue.
  
## Step 1 : Prepre a Kapua Instance (in Docker)
You need Docker on your computer to continue.

If using CloudCat instances (provisioned in step 0), execute the following commands to setup Java and Docker:

```
## Include java in $JAVA_HOME and $PATH
echo "export JAVA_HOME=/usr/java" >> ~/.bash_profile && echo "export PATH=$JAVA_HOME/bin:$PATH" >> ~/.bash_profile && source ~/.bash_profile

## Install Docker
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum-config-manager --disable docker-ce-edge

yum install -y docker-ce

systemctl start docker
```

If using a non-CloudCat environment where Docker is not installed, please search for the applicable setup guide:
https://www.google.de/search?q=setup+docker&oq=setup+docker+&aqs=chrome..69i57j0l5.6639j1j7&sourceid=chrome&ie=UTF-8



Starting Kapua requires 5 steps: 
https://www.eclipse.org/kapua/getting-started.php

```
docker run -td --name kapua-sql -p 8181:8181 -p 3306:3306 kapua/kapua-sql:0.3.2

docker run -td --name kapua-elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:5.4.0 -Ecluster.name=kapua-datastore -Ediscovery.type=single-node -Etransport.host=_site_ -Etransport.ping_schedule=-1 -Etransport.tcp.connect_timeout=30s

docker run -td --name kapua-broker --link kapua-sql:db --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 1883:1883 -p 61614:61614 kapua/kapua-broker:0.3.2

docker run -td --name kapua-console --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 8082:8080 kapua/kapua-console:0.3.2

docker run -td --name kapua-api --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 8081:8080 kapua/kapua-api:0.3.2
```

Access the Kapua Web-UI: http://127.0.0.1:8082

## Step 2 : Prepare a Kura Instance (in Docker)
The dockerized Kura setup is available here: https://github.com/ctron/kura-emulator

```
docker run -ti -p 8083:8080 ctron/kura-emulator
```

Access the Kura Web-UI: http://127.0.0.1:8083

## Step 3 : Setup the "northbound data flow"
### 3.1 Configure "CloudService" in Kura 

### 3.2 Configure Kapua-Kafka-Bridge

### 3.3 Create data ingestion pipeline to CDH

## Step 4 : Simulate a Sensor on Kura

```

 
``` 



