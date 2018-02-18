# IoT-Workshop-2018

The goal of the workshop is:

Enable a data flow from an IoT Edgedevice via Kapua and Kafka to CDH.
Sensordata should be queried via Impala in HUE. This also allows the 
integration of an BI tool using th Impala JDBC connectore.

# Short Version: less than 2 hours 

## Step 1 : Prepre a Kapua Instance
You need Docker on your computer to continue.

If Docker is not installed, please search for the right setup guide:
https://www.google.de/search?q=setup+docker&oq=setup+docker+&aqs=chrome..69i57j0l5.6639j1j7&sourceid=chrome&ie=UTF-8

Starting Kapua requires 5 steps: 
https://www.eclipse.org/kapua/getting-started.php

```
docker run -td --name kapua-sql -p 8181:8181 -p 3306:3306 kapua/kapua-sql:0.3.2

docker run -td --name kapua-elasticsearch -p 9200:9200 -p 9300:9300 elasticsearch:5.4.0 -Ecluster.name=kapua-datastore -Ediscovery.type=single-node -Etransport.host=_site_ -Etransport.ping_schedule=-1 -Etransport.tcp.connect_timeout=30s

docker run -td --name kapua-broker --link kapua-sql:db --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 1883:1883 -p 61614:61614 kapua/kapua-broker:0.3.2

docker run -td --name kapua-console --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 8080:8080 kapua/kapua-console:0.3.2

docker run -td --name kapua-api --link kapua-sql:db --link kapua-broker:broker --link kapua-elasticsearch:es --env commons.db.schema.update=true -p 8081:8080 kapua/kapua-api:0.3.2
```

## Step 2 : Prepare a CDH Cluster (using Cloud-Cat)



## Step 3 : Setup the "northbound data flow"
### 3.1 Configure "CloudService" in Kura 

### 3.2 Configure Kapua-Kafka-Bridge

### 3.3 Create data ingestion pipeline to CDH


