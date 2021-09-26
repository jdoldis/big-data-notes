---
title: Kafka
description: Kafka
sequence: 05
---

{% include toc.html %}

## Theory

Kafka can be used to reliably distribute messages between any source and target system in real time, be that production services or offline analytics processing. For example:
- A bank could publish a stream of transactions to be consumed by a fraud detection system.
- A logistics company could publish a stream of product coordinates to facilitate live location tracking.
- A video streaming platform could publish a stream of user interactions with their website to be consumed by a recommendation engine. 
- A job platform could publish a stream of job ad views to be consumed by an analytics service.
- A hotel company could publish a stream of bookings to be fulfilled.

Kafka is especially relevant now because of the explosion of specialised data systems. Applications make use of SQL databases, key value stores, document databases etc. as primary data stores. Then there is a whole class of derived data stores such as search indexes, data lakes, data warehouses, recommendation engines, monitoring aplications etc. Designating one service to act as a central nervous system of data avoids tight coupling of these systems.

In addition to decoupling, if all company data is fed to Kafka, services can access any of these streams with no extra configuration. This is a big win, as traditionally access to an organisation's complete suite of data would only be possible via batch solutions such as data warehouses.

Kafka does have some competitors in traditional messaging systms such as RabbitMQ. One thing that sets Kafka apart is that it provides storage capabilities. This has made event streams replayable, allowing data to be read flexibly and recovering from errors much simpler.

### The log

Kafka is a centralised, distributed log of messages, where each message in the log has an offset. This log is time ordered and append only. Conceptually, a log can be thought of as a database and a message a database row.

### Clients

There are two types of clients in Kafka: producers and consumers. Producers append messages to the log while consumers read messages from the log. When a producer sends a message to Kafka it does not specify the recipient of the data. Rather, the message is received by all consumers who have subscribed to the stream i.e all interactions are done via the broker, Kafka.

### Topics
When a producer publishes a message to Kafka it does so to a specific topic. Similarly, consumers read from a specific topic. Each message in a topic has a unique offset. Consumers record the offset that they have read up to to ensure that in the event of failure, a new consumer can begin reading messages where the last left off. These offsets are stored in a special Kafka topic. The process of a consumer updating the offset they have read up to is called committing. 

This is one of the main benefits of Kafka. Because it's asynchronous consumers can come up and down as they like, reading data at intervals they choose.

### Partitions
Kafka scales horizontally by splitting up the messages in a topic into partitions, which can be distributed among multiple brokers. There is no guarantee of time based ordering across partitions. For this reason producers can also specify a key when writing messages. Messages with the same key will be stored in the same partition, which guarantess ordering among those messages. If no key is specified messages are distributed evenly among partitions.

Partitions are replicated across a configurable number of brokers for high availability. One of these brokers is the partition leader, meaning it receives and serves all data for that partition. The other brokers simply stay in sync with the leader. 

When a producer writes a message to Kafka it can chose the number of replicas that need to receive the message before the message is considered to be successfully written. The choice is a tradeoff between reliability and speed:
- acks=0: message considered written as soon as it is sent over the network
- acks=1: message considered written when the leader receives it
- acks=all: message considered written when the leader and all in sync replicas receive it

Note that for consumers, a message is only available to be read once all in-sync replicas have received it.

### Consumer Goups
Consumers can be part of a group that works to read a stream of data from Kafka. This is important to ensure that the rate of consumption keeps up with that of production. Note the maximum number of consumers in a group is equal to the number of partitions.

When one consumer in a group goes down it triggers a rebalance. Consumers in the group are assigned new partitions to read from and they use the last committed offset for this. If offsets aren't committed at appropriate times, messages will be either missed or duplicated.

### Kafka Connect
There are two ways to read/write data with Kafka:
1. Traditional consumer/producer APIs
1. Kafka Connect

The original APIs require application code to be modified. Where this is not possible Kafka connect, which runs on it's own cluster,  can be used. Connect plugins for common datastores are usually pre built (e.g. Kafka to S3), but even if not the Connect API can still be used to write Kafka clients.

### Log compaction
Due to space limitations Kafka likely can't store all event streams forever. The log can be cleaned up in a few different ways:
- Space based: Clean up when the log reaches a certain size.
- Time based: Clean up when a message reaches a certian age.
- Log compaction: Discard messages where the message's key has a more recent value in the log. i.e. intermediate states are thrown away. This means that a snapshot of the producer's data system is retained.

### Log/Stream duality

For fault tolerance, databases already use logs to record all changes they make to records. Tables in a database are just a projection of this underlying changelog. Viewed this way, tables and event streams are not so different. A stream of events can be reduced into a current state and shown in a table. Similarly, a table can record all changes made to create an event stream. This duality makes it easy to move data from traditional database tables into Kafka. The process is Change Data Capture (CDC), and involves pushing the table's changelog into Kafka.


### Stream Processing
A stream of data, known as a topic in Kafka, is data that is continuously generated / unbounded. Stream processing involves incrementally reading this data, doing something and then emitting it. The application performing these operation is a stream processor, and Kafka has builtin support for such applications via Kafka Streams.

The Kafka Streams API provides builtin abstractions representing the stream/table duality discussed in the previous section. A ```KTable``` is used to represent streams where the most recent record for a key it's current state. Each record in a stream represented by a ```KTable``` is considered to be an ```UPDATE```. An example of this would be a stream of user email addresses, where the key is ```userId``` and the value ```email```. In contrast, a ```KStream``` is used to represent streams where all records for a given key need to be considered to compute it's current state. Each record is considered to be an ```INSERT```. An example of this would be caclulating bank balance from a feed of transactions.

These abstractions are used to process data and manage state. State needs to be maintained for many use cases including:
- aggregating messages in a stream, perhaps over a defined window of time.
- enriching messages by joining with data from another stream.

Using Kafka's APIs to manage this state provides builtin fault tolerance. The state is persisted to disk and additionally to an internal Kafka changelog topic.

If operating on data over a given period of time, that window of time needs to be clearly defined in terms of its size, how often to move it (the advance interval) and how late arriving events should be handled. The handling of late arriving events is generally straightforward because Kafka Streams apps write aggregation results to a results topic. This means that results can be simply overridden by writing to the results topic with the same key.


## Local Examples

### Installation

1. Download the Kafka tarball from the [Kafka downloads page](https://kafka.apache.org/downloads)
1. Unpack it:
```
sudo tar xzvf kafka*.tgz -C /opt
cd /opt
sudo mv kafka* kafka
```
1. Update environment variables:
```
export KAFKA_HOME=/opt/kafka
export PATH=$PATH:$KAFKA_HOME
```
1. Start up Zookeeper and Kafka:
```
zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
kafka-server-start.sh $KAFKA_HOME/config/server.properties
```

### Command Line Tools
The Kafka bin directory is full of scripts that make working with Kafka easy:
- Create a topic: ```kafka-topics.sh --create --topic mycooltopic --bootstrap-server localhost:9092```
- Write messages to the topic: ```kafka-console-producer.sh --topic mycooltopic --bootstrap-server localhost:9092```. After running this messages are entered via stdout.
- Read messages from the topic: ```kafka-console-consumer.sh --topic mycooltopic --from-beginning --bootstrap-server localhost:9092```
- Read messages with key, value and timestamp from the topic: ```kafka-console-consumer.sh --topic mycooltopic --from-beginning --property print.key=true --property key.separator="-" --property print.timestamp=true --bootstrap-server localhost:9092```
- Show the size of the topic's partitions: ```kafka-log-dirs.sh --describe --topic-list mycooltopic --bootstrap-server localhost:9092```
- Show number of partitions/replication factor of the topic: ```kafka-topics.sh --describe --topic mycooltopic --bootstrap-server localhost:9092```
- Get the last offset in the topic: ```kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mycooltopic```
- List all Kafka topics: ```kafka-topics.sh --list --bootstrap-server localhost:9092```

### Consumers and Producers
There are plenty of code examples for consumers and producers, for example [in Java](https://docs.confluent.io/platform/current/tutorials/examples/clients/docs/java.html). 

### Kafka Connect
#### Standalone Mode
The ```connect-standalone.sh``` script can be used to execute a worker in standalone mode. For example a file source connector can be run with ```connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties $KAFKA_HOME/config/connect-file-source.properties```. This will write the contents of the file ```test.txt``` to the Kafka topic ```connect-test```, as per the configuration files passed to the script. Such a connector would be using, for example, to stream application logs into Kafka.

A file source sink is also easy enough: ```connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties $KAFKA_HOME/config/connect-file-sink.properties```. This will write the message in ```connect-test``` to a file named ```test.sink.txt```.

Note this command will fail if the source worker cluster is still running as the default REST API port is already taken. There are 2 possible resolutions to this:
1. Run two connectors on one worker: ```connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties $KAFKA_HOME/config/connect-file-source.properties $KAFKA_HOME/config/connect-file-sink.properties```
1. Change the api port of the second worker by specifying the ```rest.port``` property in the standalone properties file.

The REST docs can be found [here](https://docs.confluent.io/platform/current/connect/references/restapi.html)

Custom plugins can also be used. For example, say you want to move data from Kafka to S3.
1. Download the [s3 sink connector](https://www.confluent.io/hub/confluentinc/kafka-connect-s3).
1. Unzip and move the folder to ```/opt/connectors```.
1. Set ```plugin.path``` to ```/opt/connectors``` in the standalone worker properties file.
1. Ensure ```~/.aws/credentials``` exists.
1. Create an s3 sink properties file:
```
name=s3-sink
connector.class=io.confluent.connect.s3.S3SinkConnector
tasks.max=1
topics=connect-test
flush.size=3
s3.bucket.name=kafka-connect-s3-sink-example
s3.region=eu-west-1
storage.class=io.confluent.connect.s3.storage.S3Storage
format.class=io.confluent.connect.s3.format.json.JsonFormat
```
1. Start the worker with this file: ```connect-standalone.sh $KAFKA_HOME/config/connect-standalone.properties ./s3-sink.properties```
1. Check s3 for the data

#### Distributed Mode
When running distributed workers, the REST API needs to be used for configuration. It's is also available in standalone mode, but connectors can be created by simply passing properties files to the script.

Useful REST API endpoints include:
- Test connectivity: ```curl localhost:8083``` (or whatever ```rest.port``` is set to).
- List connectors running on the cluster: ```curl localhost:8083/connectors```
- Show details of a specific connector: ```curl localhost:8083/connectors/<connector_name>```. The connector name is returned by the ```connectors``` endpoing, and in this case is curl ```local-file-source```.
- Show the tasks a connector is running: ```curl localhost:8083/connectors/<connector_name>/tasks/```.
- Show the plugins available: ```curl localhost:8083/connector-plugins```.

Adding/removing connectors is also not too hard. For example, start a distributed worker: ```connect-distributed.sh $KAFKA_HOME/config/connect-distributed.properties```. To add a connector:
```
curl \
    localhost:8083/connectors \
    -H "Content-Type: application/json" \
    -X POST \
    --data-binary @- << EOF
{
    "name": "file-source-connector",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": "1",
        "file": "test.txt",
        "topic": "connect-test"
    }
}
EOF
```

It can be removed with ```curl -X DELETE http://localhost:8083/connectors/file-source-connector```.

### Kafka Streams
There are plenty of Kafka streams code examples [on Confluent](https://docs.confluent.io/platform/current/streams/code-examples.html). Kafka also ships with a wordcount streams example, which can be run by executing ```kafka-run-class org.apache.kafka.streams.examples.wordcount.WordCountDemo```. This reads from the topic ```streams-plaintext-input``` and writes to ```streams-wordcount-output```.
