### **Apache Kafka Cheat Sheet**

#### **Introduction**
Apache Kafka® is an **open-source distributed event streaming platform** used for building **real-time data pipelines** and streaming applications. Kafka is **horizontally scalable**, **fault-tolerant**, and **highly durable**.

This guide provides an in-depth look at Kafka's **architecture**, **core concepts**, and **commonly used commands** with detailed explanations and examples.

---

#### **1. Key Concepts**
##### **1.1 Clusters**
- A **Kafka Cluster** is a collection of **brokers (servers)** working together.
- Provides **fault tolerance, scalability, and high throughput**.
- Clusters handle **millions of messages per second** in distributed systems.

##### **1.2 Topics**
- A **topic** is a logical channel where messages are **produced and consumed**.
- Each topic is **split into partitions** for parallel processing.
- Topics are **multi-subscriber**, meaning multiple consumers can read from them.

**Rules for Naming Topics:**
1. Topic names should **only contain** letters (`a-z`, `A-Z`), numbers (`0-9`), dots (`.`), underscores (`_`), and hyphens (`-`).
2. Topic names should be **descriptive** and meaningful.
3. **Avoid special characters** like `@`, `#`, `!`, `*`, as Kafka does not support them.

**Examples of Topic Names:**
```
# Valid topic names
customer_orders
logs.application-errors
user_activity

# Invalid topic names (containing special characters)
customer@orders  # Invalid '@'
logs#errors      # Invalid '#'
```

---

#### **2. Brokers**
- A **broker** is a Kafka server that stores and serves messages.
- Kafka brokers manage:
  - **Topic partitions**
  - **Message replication**
  - **Data storage & retrieval**
- Brokers are part of a **Kafka cluster** and work together.

---

#### **3. Producers**
- Producers send (publish) messages to **Kafka topics**.
- Messages are assigned to **partitions** based on:
  - **Round-robin (default)**
  - **Key-based partitioning** (Ensures messages with the same key go to the same partition)

**Example: Writing messages to a Kafka topic**
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic customer_orders
```
Type messages and press **Enter** to send them.

---

#### **4. Consumers**
- Consumers **read messages** from Kafka topics.
- Consumers belong to **consumer groups**, allowing **parallel processing**.

**Example: Reading messages from a topic**
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic customer_orders --from-beginning
```
This will print **all past and new messages** from the `customer_orders` topic.

---

#### **5. Partitions**
- Topics are split into **partitions** to enable **parallel consumption**.
- Each partition is **ordered** and messages are assigned an **offset**.
- Partitions allow **horizontal scaling**.

**Example of a topic with 3 partitions:**
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic logs
```

**Rules for Partitions:**
1. More partitions = **better parallelism**.
2. Cannot **reduce** partitions, only **increase** them.
3. Messages are assigned partitions **based on key hashing** or round-robin.

---

#### **6. Kafka Connect**
Kafka Connect is used to **stream data** between Kafka and **external data systems** like **databases, file systems, and cloud storage**.

**Example: Running a Kafka Connect Worker**
```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties
```

---

#### **7. Kafka CLI Commands**
##### **7.1 Starting Kafka Components**
```
# Start Zookeeper
zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka Server
kafka-server-start.sh config/server.properties
```

---

##### **7.2 Managing Topics**
###### **List Topics**
```
bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

###### **Describe a Topic**
```
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic customer_orders
```

###### **Create a Topic (3 Examples)**
```
# Create a topic with 1 partition and replication factor of 1
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic customer_orders

# Create a topic with 3 partitions and replication factor of 2
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 3 --topic logs

# Create a topic for real-time analytics with 5 partitions
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 5 --topic analytics_stream
```

###### **Delete a Topic**
```
bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic old_topic
```

###### **Increase Partitions**
```
bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic logs --partitions 5
```
**⚠️ Note:** Kafka **does not** allow **decreasing** partitions!

---

##### **7.3 Managing Messages**
###### **Find Number of Messages in a Topic**
```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic logs --time -1 --offsets 1 | awk -F ":" '{sum += $3} END {print sum}'
```

###### **Get Earliest and Latest Offsets**
```
# Earliest offset (first message)
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic logs --time -2

# Latest offset (last message)
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic logs --time -1
```

---

##### **7.4 Consumer Groups**
###### **List Consumer Groups**
```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

###### **Describe a Consumer Group**
```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my_consumer_group
```

---

##### **7.5 Using Kafkacat**
###### **Read Last 5 Messages from a Topic**
```
kafkacat -C -b localhost:9092 -t logs -p 0 -o -5 -e
```

---

#### **8. Advanced Notes**
1. **Kafka Retention Policies**:
   - Kafka can retain messages **forever**, for a set **time period**, or until reaching a **size limit**.
   - Configure **log retention** with:
   ```
   bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic logs --config retention.ms=604800000  # 7 days
   ```

2. **Monitoring Kafka**:
   - Use Kafka's built-in tools or third-party **monitoring solutions** like **Confluent Control Center, Prometheus, or Grafana**.

3. **Kafka Streams API**:
   - Used for **real-time data processing** within Kafka itself.
   - Helps build **event-driven applications**.

---

#### **9. References**
For more details, check out:
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Redpanda Kafka Guide](https://redpanda.com/guides/kafka-tutorial)
