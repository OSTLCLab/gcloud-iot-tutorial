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
4. Create an RS265 key pair in PEM format for your Device: https://cloud.google.com/iot/docs/how-tos/credentials/keys 
ToDo: Add process for Windows
5. Create a new device: Select the Divices Tab and choose "Create a Device" 
```
Device ID : pde-module-0001
Authentication : Enter manually
Public key value : Copy and Paste the content of the pulic key file
Stackdriver Logging : Debug
```
Remark: The device we just created reflects the "digital twin" of our real device in the cloud. Later the real device will send data to its digital twin. The cloud side need to be absolutely sure, that this data was sent by the one and only real device. This is done with help of the key pair we created: Every data package (in our case JSON Web Tokens, JWT) will be signed on the real device with the help of its private key (only known by the device). The digital twin in the cloud can verify the identiy of the sender with the help of the public key (see Signature use case of the security lecture)

### Setup of Device (Node-Red)
1. Install gcloud
Now, your divce is talking to the Cloud

## Vizalize Data
