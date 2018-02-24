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

Access the Kapua Web-UI: http://YOUR_KAPUA_HOST:8082

Check the docker containers:

```
docker ps
```

## Step 2 : Prepare a Kura Instance (in Docker)
The dockerized Kura setup is available here: https://github.com/ctron/kura-emulator

Please repeat the docker installation on a different host and run the KURA docker container in this host.
Both, Kura and Kapua have MQTT brokers using the port 1883, thus we us two different host to avoid collisions and confusion.

```
docker run -ti -p 8083:8080 ctron/kura-emulator
```

Access the Kura Web-UI: http://YOUR_KURA_HOST:8083

## Step 3 : Setup the "northbound data flow"
### 3.1 Configure "CloudService" in Kura 

### 3.2 Configure Kapua-Kafka-Bridge

### 3.3 Create data ingestion pipeline to CDH

#### 3.3.1 Create Metrics View for Impala Table for HUE and Tableau 
``` 
CREATE VIEW IF NOT EXISTS iiot.telemetry_view as
SELECT
spine.millis
,spine.date_value
,spine.datetime_value
,spine.id
,cast(t1.value as int) as 'acceleration_x'
,cast(t2.value as int) as 'acceleration_y'
,cast(t3.value as int) as 'acceleration_z'
,cast(t4.value as int) as 'gyro_x'
,cast(t5.value as int) as 'gyro_y'
,cast(t6.value as int) as 'gyro_z'
,cast(t7.value as int) as 'mag_x'
,cast(t8.value as int) as 'mag_y'
,cast(t9.value as int) as 'mag_z'
,cast(t10.value as int) as 'temperature'
,cast(t11.value as int) as 'humidity'
,cast(t12.value as int) as 'pressure'
,cast(t13.value as int) as 'light'
FROM
(SELECT DISTINCT
millis
,from_unixtime((millis DIV 1000), 'yyyy/MM/dd') as date_value
,from_unixtime((millis DIV 1000)) as datetime_value
,id
FROM iiot.telemetry
WHERE (millis DIV 1000) BETWEEN (unix_timestamp(now())-300) and unix_timestamp(now())) as spine
left outer join iiot.telemetry t1 on (spine.millis = t1.millis and
spine.id = t1.id)
left outer join iiot.telemetry t2 on (spine.millis = t2.millis and
spine.id = t2.id)
left outer join iiot.telemetry t3 on (spine.millis = t3.millis and
spine.id = t3.id)
left outer join iiot.telemetry t4 on (spine.millis = t4.millis and
spine.id = t4.id)
left outer join iiot.telemetry t5 on (spine.millis = t5.millis and
spine.id = t5.id)
left outer join iiot.telemetry t6 on (spine.millis = t6.millis and
spine.id = t6.id)
left outer join iiot.telemetry t7 on (spine.millis = t7.millis and
spine.id = t7.id)
left outer join iiot.telemetry t8 on (spine.millis = t8.millis and
spine.id = t8.id)
left outer join iiot.telemetry t9 on (spine.millis = t9.millis and
spine.id = t9.id)
left outer join iiot.telemetry t10 on (spine.millis = t10.millis and
spine.id = t10.id)
left outer join iiot.telemetry t11 on (spine.millis = t11.millis and
spine.id = t11.id)
left outer join iiot.telemetry t12 on (spine.millis = t12.millis and
spine.id = t12.id)
left outer join iiot.telemetry t13 on (spine.millis = t13.millis and
spine.id = t13.id)
WHERE
t1.metric = 'acc_x'
and t2.metric = 'acc_y'
and t3.metric = 'acc_z'
and t4.metric = 'gyro_x'
and t5.metric = 'gyro_y'
and t6.metric = 'gyro_z'
and t7.metric = 'mag_x'
and t8.metric = 'mag_y'
and t9.metric = 'mag_z'
and t10.metric = 'temperature'
and t11.metric = 'humidity'
and t12.metric = 'pressure'
and t13.metric = 'light'
ORDER BY millis desc
--limit 100;
```
## Step 4 : Measure the Temperature using the Raspberry PI

### 4.1 Read the Sensor data using Python

```

 
``` 
### 4.2 Simulate the temperature and send values to KAPUA

```

 
``` 
### 4.3 Send the temperature values to KURA

```

 
``` 




