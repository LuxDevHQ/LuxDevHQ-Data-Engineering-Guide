# Tuesday: Kafka Producer and Consumer (Lab)

## Objectives

Set up a basic Kafka environment with:

* A **Producer** that sends messages to a topic
* A **Topic** to hold the messages
* A **Consumer** that reads messages from the topic

## Step-by-Step Setup

### 1. Ensure Java is Installed

Kafka requires Java.

```bash
java -version
```

If not installed:

```bash
sudo apt install openjdk-11-jdk -y
```

### 2. Download and Extract Kafka

Kafka is distributed as a compressed archive (`.tgz` file). Here’s how to download and unpack it:

```bash
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.13-3.6.1.tgz
```

 `wget` is a command-line tool used to download files from the web.

 This command fetches Kafka version **3.6.1** built for **Scala 2.13** (Kafka is written in Scala and Java).

 You can always check for the latest version at [Kafka downloads](https://kafka.apache.org/downloads).

Next, extract the downloaded archive:

```bash
tar -xzf kafka_2.13-3.6.1.tgz
```

* `tar` is used to extract files.
  * `-xzf` means:
  
    * `x`: extract
    * `z`: decompress gzip
    * `f`: specify the file name

Then, delete the downloaded archive to free up space:

```bash
rm kafka_2.13-3.6.1.tgz
```

Rename the extracted folder to something simpler:

```bash
mv kafka_2.13-3.6.1 kafka
```

Change into the Kafka directory:

```bash
cd kafka
```

This is Kafka’s **home directory** where all config files, scripts, and binaries live.

You’ll run all Kafka-related commands from here.

#### Kafka Folder Structure Overview

| Folder / File       | Purpose                                          |
| ------------------- | ------------------------------------------------ |
| `bin/`              | Scripts to start Kafka, ZooKeeper, and CLI tools |
| `config/`           | Configuration files for Kafka and ZooKeeper      |
| `libs/`             | Kafka and ZooKeeper libraries                    |
| `logs/`             | Kafka server logs during runtime                 |
| `LICENSE`, `NOTICE` | Legal/license information                        |

Kafka is now ready to be started.

### 3. Start ZooKeeper and Kafka

Start ZooKeeper (in one terminal):

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

Start Kafka (in another terminal):

```bash
bin/kafka-server-start.sh config/server.properties
```
### 4. Create a Kafka Topic

```bash
bin/kafka-topics.sh --create \
  --topic test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

-  **Partitions**: Allow parallelism and scalability.
-  **Replication factor**: Number of copies for fault tolerance.

### 5. Start a Producer

```bash
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
```

Type messages and press Enter to send them to the topic.

### 6. Start a Consumer

```bash
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

Messages sent by the producer will appear here.

## Topic Management Commands

**List topics**

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

**Describe a topic**

```bash
bin/kafka-topics.sh --describe --topic test-topic --bootstrap-server localhost:9092
```

**Delete a topic**

```bash
bin/kafka-topics.sh --delete --topic test-topic --bootstrap-server localhost:9092
```

**Add more partitions**

```bash
bin/kafka-topics.sh --alter --topic test-topic --partitions 3 --bootstrap-server localhost:9092
```

> Note: You can **only increase** partitions, not reduce them.

## Summary

* Kafka and ZooKeeper are up and running
* Topic created
* Messages produced and consumed
* Topics managed using CLI tools


