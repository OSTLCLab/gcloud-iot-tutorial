# gcloud IoT Tutorial
A very simple Tutorial to connect an IoT Device to gcloud over MQTT.
This tutorial is part of the a course for mechanical engineers to get familiar with the cloud side of the IoT stack.

## Intial Situation
* Studneds have developed a simple machine that sorts Lego bricks. The machine has sensor(s) to control it success/failure rate in picking and placing the bricks to the corect position (Machine is built with Lego and Arduino).
* The embedded software on the machine is able communicate with Raspberry PI via a proprietary protocol (Textmessages over BLE).
* The Raspberry Pi acts as a gateway between the factory where machine is operating and the cloud where the success/failure rates of all machins are tracked by the producer of the machine to further improve it.
* Node-red is running on the Raspberry Pi

## Goals of the Tutorial

Overall Goal: 
* Connect Node-red to gCloud over MQTT and store success/failure statistics of a device
* Get familiar with cloud plattforms
* Get familier with basic device management
* Get faimliar with basic security/authenitcation concepts

Steps to be done
* Open an account on gCloud
* Create a virtual device on IoT Core
* Connect the real device to IoT Core
* Act on incomming data of devices
* Store the incomming data into a database 
* Vizualize data

## Step by step tutorial

### Open an Account on gCloud
1. Create an Account for Google cloud, using your voucher.
2. Login to the console and make sure you are working in the correct project

### Setup of IoT Core
1. On the left navigation bar select IoT Core (its located in the "Big Data" section
2. Activate the IoT Core service
3. Create a regisitry. The regisitry groups together a fleet of devices that share common properties and rules e.g. for a particular application.
```
Name : pde-pickandplace-modules
Region : europe-west1
Protocol : MQTT and HTPP
Default telemetry topic : Create new topic 'default'
Device state topic : Create new topic 'status'
Stackdriver Logging : None
```
4. On your deivce (Raspberry PI): Open the console and create an RS265 key pair with self-sigend certificate in PEM format: https://cloud.google.com/iot/docs/how-tos/credentials/keys 

```
openssl req -x509 -nodes -newkey rsa:2048 -keyout pde_rsa_private.pem -days 1000000 -out pde_rsa_cert.pem -subj "/CN=unused"
```

5. Create a new device: Select the Divices Tab and choose "Create a Device" 
```
Device ID : pde-module-0001
Authentication : Enter manually
Public key format: RS256_X509
Public key value : Copy and Paste the content of the pde_rsa_cert.pem file
Stackdriver Logging : Debug
```
Remark: The device we just created reflects the "digital twin" of our real device in the cloud. Later, the real device will send data to its digital twin. The cloud side needs to be absolutely sure that this data was sent by the one and only real device. This is achieved by help of the key pair we created: Every data package (in our case JSON Web Tokens, JWT) will be signed on the real device with the help of its private key (only known by the device). The digital twin in the cloud can verify the identity of the sender with the help of the public key (see Signature use case of the security lecture)

### Setup of Device (Node-Red)
The node-red instance reflects the real device in our example.
1. Start node-red on your device (or for testing on your local machine)
2. Open node-red in the browser (usually: http://localhost:1880)
3. Open a new flow and import the follwing nodes:
```
[{"id":"a6082cec.d600e","type":"inject","z":"a500d8d8.5e24f8","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":200,"y":440,"wires":[["fffc4044.d99d3"]]},{"id":"fffc4044.d99d3","type":"function","z":"a500d8d8.5e24f8","name":"","func":"msg.payload = {\n    timestamp : msg.payload,\n    temperature : 24\n}\nreturn msg;","outputs":1,"noerr":0,"x":390,"y":440,"wires":[["902471b5.02b21","d06110d8.8af81"]]},{"id":"902471b5.02b21","type":"mqtt out","z":"a500d8d8.5e24f8","name":"","topic":"","qos":"","retain":"","x":610,"y":440,"wires":[]},{"id":"d06110d8.8af81","type":"debug","z":"a500d8d8.5e24f8","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":630,"y":500,"wires":[]}]
```
This flow sends a simple JSON object over the MQTT protocol

4. Doubleclick the mqtt node and add a new mqtt broker:
```
The Server of Cloud IoT Core is: mqtt.googleapis.com:8883
Add new TSL-Configuration:
Upload the pde_rsa_cert.pem and the pde_rsa_private.pem files (rest can stay blank)
Edit the details of the device you just created on IoT Core

```
5. Deploy the flow

This flow sends a simple JSON object over the MQTT protocol. If everything works fine, you finde a log entry (go to the details of the device in IoT Core and click on "View logs").

Now, your divce is talking to the Cloud

## Store data from IoT Devices
To store the data we sent form the device we need to act on the MQTT messages and store the data into a database, in our case into Google's BigQuery.

### Create a Datatable in BigQuery
Our data will be stored in a big data table.

1. Navigate to "BigQuery" in the BigData section of the main navigation in goolge consle
2. Select the project on the left side and then click on "Create Dataset"
```
Name : pde_module
Default Location: EU
```
3. Select the dataset we just created and click on "Create Table"
```
Table name : raw_data
Add fields:
Field name  Type      Mode
device_id   STRING    NULLABLE	
timestamp   TIMESTAMP NULLABLE	
success     BOOLEAN   NULLABLE	
temperature FLOAT     NULLABLE

```



### Create a Temporary Storage
The template we will use to push our IoT Data to BigQuery needs a temporary file store.

1. Navigate to "Storage" in the Storage section of the main navigation in goolge consle
2. Create a new bucket "pde-temp"
3. Create a new folder in this bucket "iot-data"

## Vizualize
Go to datastudio (its not part of the Google Cloud console)
https://datastudio.google.com/navigation/reporting


