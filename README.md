# MongoDB, Kafka Connect, MongoDB Connector for Apache Kafka - Source and Sink example

This demo leverages docker and sets up the following infrastructure:

A MongoDB Cluster (3 node replica set)
Kafka Connect
Kafka Broker
Zookeeper
Kafka Rest API
Kafka Schema Registry

There is a file called RUN.SH.  This file issues the docker-compose up as well as configures the local MongoDB cluster as a source and configures the cluster passed as a parameter as a sink.

There is a python app that will randomly generate ficticious companies and generate stock prices every second for as long as the app is running.  These data are writen to the MongoDB cluster locally, since the MongoDB Connector is setup as a sink it will grab the values and push it into a topic on the Kafka cluster.

The Sink will take the data in the topic and push it out to the cluster that is passed as a parameter on the RUN.SH command.  For demo purposes we are using a MongoDB Atlas cluster.

## Requirements
  - Docker 18.09+
  - Docker compose 1.24+
  - [Kafkacat](https://github.com/edenhill/kafkacat) (optional)
  - Python3 with pymongo
  
## Running the demo
### 1. Download/Clone the docker files from the GitHub repository

[https://github.com/RWaltersMA/mongo-source-sink.git](https://github.com/RWaltersMA/mongo-source-sink.git)

### 2. Copy the Atlas Connection String

If you do not have a MongoDB Atlas cluster for use as the final destination, [follow these instructions](https://docs.atlas.mongodb.com/getting-started/).

Just creating the cluster is not enough to run the demo.  You will need to define a database user for use by the Kafka Connector to connect to the MongoDB Atlas cluster.  You will also have to whitelist the IP address of the docker host.

If you have not created a database user for the Kafka Connector:

Select, “Database Access” from the Atlas menu and click the “Add new User” button.  

Provide a username and password and select, “Read and write to any database”.  Remember the password.

If your docker host is not whitelisted:
Click, “Network Access” from the Atlas menu and click, “Add IP Address”.  Here you can add your current IP address or any IP Address or block of addresses in CIDR format.  Note: If you do not know or can not obtain the IP address of the docker host you can add, “0.0.0.0” as an entry which will allow connections from anywhere on the internet.  This is not a recommended configuration.

To copy the connection string select the “CONNECT” button on your Atlas cluster then choose “Connect your application”.  Click the Copy button to copy the connection string to the clipboard.</p>

### 4. Execute the RUN.SH script passing Atlas Connection String

The demo is now ready to go just issue a `sh run.sh "<<paste in your Atlas Connection String here>>"` and the script will start the docker containers and configure the connectors.

## Running the Demo

If the RUN.SH scripts runs successfully it should will say it is ready for data. 

### 1. Run the python security generator application 

run `python3 realtime-mongo.py` in a new shell to start generating ficticuous security data.  The following are optional parameters 

-s    the number of financial symbols to generate (default is 10)
-c    mongodb connection string (default is localhost)

### 2. View the topic messages

You can confirm the source connector is working by reading messages from the Kafka Topic, "stockdata.Stocks.StockData". 

`kafkacat -b 127.0.0.1:9092  -t stockdata.Stocks.StockData`
…
"{\"_id\": {\"$oid\": \"5e307e3940bacb724265e4a8\"}, \"company_symbol\": \"ISH\", \"company_name\": \"ITCHY STANCE HOLDINGS\", \"price\": 35.02, \"tx_time\": \"2020-01-28T18:32:25Z\"}"

### 3. View the message in Atlas

The MongoDB Connector for Apache Kafka is configured as a sink connector and writes data to MongoDB Atlas (or the MongoDB cluster defined as the parameter to the RUN.SH script). Data is written to the StockData collection in the Stocks database.  To view the data in Atlas, click on "Collections" tab in your MongoDB Atlas portal to view the StockData collection.

# Troubleshooting
Most failures occur because of network connectiviity issues.  If there is a failure check the docker logs of the containers to start troubleshooting.  Most failures occur with network connectivity issues between the connect container and MongoDB Atlas.  Remember to add the appropriate IP whitelist and Username to the Altas Cluster.