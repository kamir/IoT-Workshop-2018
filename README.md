# IoT-Workshop-2018

The goal of the workshop is:

Enable a data flow from an IoT Edgedevice via Kapua and Kafka to CDH.
Sensordata should be queried via Impala in HUE. This also allows the 
integration of an BI tool using th Impala JDBC connectore.

# Short Version: less than 2 hours 

## Step 1 : Prepre a Kapua Instance
You need Docker on your computer to continue.

If Docker is not installed, please follow this guide:
https://www.google.de/search?q=setup+docker&oq=setup+docker+&aqs=chrome..69i57j0l5.6639j1j7&sourceid=chrome&ie=UTF-8

Starting Kapua requiires 5 steps: 
https://www.eclipse.org/kapua/getting-started.php

## Step 2 : Prepare a CDH Cluster (using Cloud-Cat)



## Step 3 : Setup the "northbound data flow"
### 3.1 Configure "CloudService" in Kura 

### 3.2 Configure Kapua-Kafka-Bridge

### 3.3 Create data ingestion pipeline to CDH


