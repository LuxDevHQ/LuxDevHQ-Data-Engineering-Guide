### Kafka-cheat-sheet  

Apache Kafka® serves as an open-source distributed streaming platform. Similar to other distributed systems, Kafka boasts a complex architecture, which may pose a challenge for new developers. Setting up Kafka involves navigating a formidable command line interface and configuring numerous settings. In this guide, I will provide insights into architectural concepts and essential commands frequently used by developers to initiate their journey with Kafka.

#### Key Concepts:

- **Clusters**: Group of servers working together for three reasons: speed (low latency), durability, and scalability.
- **Topic**: Streams of records that Kafka organizes data into.
- **Brokers**: Kafka server instances that store and replicate messages.
- **Producers**: Applications that write data to Kafka topics.
- **Consumers**: Applications that read data from Kafka topics.
- **Partitions**: Divisions of a topic for scalability and parallelism.
- **Connect**: Kafka Connect manages the Tasks. Note that the Connector is only responsible for generating the set of Tasks and indicating to the framework when they need to be updated.

The easiest way to run Kafka clusters is to use **Confluent Cloud**, which is the official maintainer of Apache Kafka and provides different libraries for writing producers, consumers, and schema registry.

#### Summary

1. **Apache Kafka**: A distributed streaming platform.

The Kafka CLI is a powerful tool. However, the user experience can be challenging if you don’t already know the exact command needed for your task. Below are the commonly used CLI commands to interact with Kafka:

#### Start Zookeeper
```sh
zookeeper-server-start config/zookeeper.properties
```

#### Start Kafka Server
```sh
kafka-server-start config/server.properties
```

### Kafka Topics

- **List existing topics**
```sh
bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

- **Describe a topic**
```sh
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic mytopic
```

- **Purge a topic**
```sh
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --config retention.ms=1000
```

or

```sh
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --delete-config retention.ms
```

- **Delete a topic**
```sh
bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic mytopic
```

- **Get number of messages in a topic**
```sh
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1 --offsets 1 | awk -F  ":" '{sum += $3} END {print sum}'
```

- **Get the earliest offset still in a topic**
```sh
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -2
```

- **Get the latest offset still in a topic**
```sh
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1
```

- **Consume messages with the console consumer**
```sh
bin/kafka-console-consumer.sh --new-consumer --bootstrap-server localhost:9092 --topic mytopic --from-beginning
```

- **Get the consumer offsets for a topic**
```sh
bin/kafka-consumer-offset-checker.sh --zookeeper=localhost:2181 --topic=mytopic --group=my_consumer_group
```

- **Read from `__consumer_offsets`**
Add the following property to `config/consumer.properties`:
```sh
exclude.internal.topics=false
```

Then run:
```sh
bin/kafka-console-consumer.sh --consumer.config config/consumer.properties --from-beginning --topic __consumer_offsets --zookeeper localhost:2181 --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter"
```

### Kafka Consumer Groups

- **List the consumer groups known to Kafka**
```sh
bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list # (old API)
```
```sh
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list # (new API)
```

- **View the details of a consumer group**
```sh
bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group <group_name>
```

### Kafkacat

- **Getting the last five messages of a topic**
```sh
kafkacat -C -b localhost:9092 -t mytopic -p 0 -o -5 -e
```

### Zookeeper

- **Starting the Zookeeper Shell**
```sh
bin/zookeeper-shell.sh localhost:2181
```

### Running Java Class

- **Run `ConsumerOffsetCheck` when Kafka server is up, there is a topic + messages produced and consumed**
```sh
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --broker-info --zookeeper localhost:2181 --group test-consumer-group
```

**Note:** `ConsumerOffsetChecker` has been removed in Kafka 1.0.0. Use `kafka-consumer-groups.sh` to get consumer group details:
```sh
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group consule-consumer-38063
```

### Kafka Server & Topics

- **Start Zookeeper**
```sh
bin/zookeeper-server-start.sh config/zookeeper.properties
```

- **Start Kafka brokers (Servers = cluster)**
```sh
bin/kafka-server-start.sh config/server.properties
```

- **Create a topic**
```sh
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

- **List all topics**
```sh
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

- **See topic details (partition, replication factor, etc.)**
```sh
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
```

- **Change partition number of a topic (`--alter`)**
```sh
bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic test --partitions 3
```

### Producer

```sh
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

### Consumer

```sh
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic test
```

- **To consume only new messages**
```sh
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test
```

### Kafka Connect

- **Standalone connectors (run in a single, local, dedicated process)**
```sh
bin/connect-standalone.sh \ config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

### Reference

[Redpanda Kafka Tutorial](https://redpanda.com/guides/kafka-tutorial)
