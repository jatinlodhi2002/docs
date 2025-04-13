---

# How to Self Host Sparrow in ubuntu

Hosting our Sparrow on Ubuntu. This guide provides step-by-step instructions for self-hosting our Sparrow API on an Ubuntu server.

```
Prerequisite
-------------------------------------------------------------
   - Ubuntu server version (22.04.2)
   - Sparrow API codebase
   - Node.js (v18.19.1)
   - MongoDB (v7.0.18)
   - Kafka (Kafka 4.0.0)
```

## Install Apache Kafka on Ubuntu
---
### Step 1: Creating a user for Kafka
The first step is to create a dedicated user to ensure that Kafka's operations do not interfere with the system's other functionalities.

Add a new user called kafka:

`sudo adduser kafka`

Next, you need to add the kafka user to the sudo group to have the necessary privileges for Kafka installation.

`sudo adduser kafka sudo`

Then, log in to the kafka account:

`su -l kafka`

The kafka user now is ready to be used.

### Step 2: Installing Java Development Kit (JDK)
Open the terminal and update the package index: 

`sudo apt update`

Install the OpenJDK 11 package:

`sudo apt install openjdk-11-jdk`

Now that you’ve installed the JDK, you can start downloading Kafka.

### Step 3: Downloading Kafka
Start by creating a folder named downloads to store the archive:

`mkdir ~/downloads`

`cd ~/downloads`

`wget "https://downloads.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz"`

Then, move to ~ and extract the archive you downloaded:

`cd ~`
`tar -xvzf ~/downloads/kafka_2.13-4.0.0.tgz`

Let’s rename the directory kafka_2.13-4.0.0 to kafka.
`mv kafka_2.13-4.0.0/ kafka/`

Now that you’ve downloaded Kafka, you can start configuring your Kafka server.

### Step 4: Configuring the Kafka server
First, start by setting the log.dirs property to change the directory where the Kafka logs are.

To do so, you need to edit the server.properties file:
`nano ~/kafka/config/server.properties`

Look for `log.dirs` and set the value to `/home/kafka/kafka-logs.`


### Step 5: Starting the Kafka server

To start the Kafka server, you need to first start Zookeeper and then start Kafka.

But, to be more efficient, you need to create systemd unit files and use systemctl instead.

Unit File for Zookeeper:
`sudo nano /etc/systemd/system/zookeeper.service`

```
[Unit]
Description=Apache Zookeeper Service
Requires=network.target                 
After=network.target                 

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties        
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
 
 Unit File for Kafka:
`sudo nano /etc/systemd/system/kafka.service`

```
[Unit]
Description=Apache Kafka Service that requires zookeeper service
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart= /home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties                            
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
Then, you can start the Kafka server:
`sudo systemctl start kafka`

Check the status:
`sudo systemctl status kafka`

Step 6: Testing the Kafka server

You can check if the Kafka server is up with netcat. By default, Kafka server runs on 9092:
`nc -vz localhost 9092`

---

## Install MongoDB Community Edition

### Step1: Import the public key.

From a terminal, install gnupg and curl if they are not already available:
`sudo apt-get install gnupg curl`

To import the MongoDB public GPG key, run the following command:

```
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```

### Step2: Create the list file
Create the list file `/etc/apt/sources.list.d/mongodb-org-7.0.list` for version Ubuntu version.

```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

### Step3: Reload the package database

Issue the following command to reload the local package database:
`sudo apt-get update`

### Step4: Install MongoDB Community Server
You can install either the latest stable version of MongoDB.
`sudo apt-get install -y mongodb-org`

```
#Start MongoDB
sudo systemctl start mongod

#Verify that MongoDB has started successfully
sudo systemctl status mongod

#You can optionally ensure that MongoDB will start following a system reboot by issuing the following command:
sudo systemctl enable mongod
```

---

## Install Node.js

Ensure that Node.js is installed on Ubuntu server. You can do this by running the following commands:

`sudo apt update`
`sudo apt install nodejs`

**Install npm & pnpm**
npm (Node Package Manager) should be installed along with Node.js. Verify its installation by running:

`npm -v`
`pnpm install`

---

## Clone Sparrow api repository

#### Clone the repository
    git clone https://github.com/sparrowapp-dev/sparrow-api.git

#### Move into the project directory
    cd sparrow-api

#### Install PNPM globally
    npm i -g pnpm

#### Rename .env.example to .env
    cp .env.example .env

#### Install dependencies
	pnpm i

#### Run App in development mode
	pnpm start:dev

Access swagger on localhost:
Go to http://localhost:{PORT}/api/docs





















