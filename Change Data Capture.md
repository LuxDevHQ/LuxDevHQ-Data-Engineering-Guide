# Change Data Capture (CDC) Learning Guide

## What is Change Data Capture ?

Change Data Capture  is a powerful technique that tracks changes—specifically inserts, updates, and deletes—in a source database and streams them to a target system in real-time or near real-time. Think of CDC as a vigilant observer that watches your database and immediately reports any changes to other systems that need to stay synchronized.

This approach ensures data consistency across different systems like data warehouses, caches, or analytics platforms without the need to process the entire dataset repeatedly. CDC serves as the backbone for data integration, real-time analytics, and maintaining up-to-date information across distributed systems.

## Why Use CDC?

Understanding the benefits of CDC helps explain why it has become essential in modern data architectures:

**Real-Time Data Synchronization**: Unlike traditional batch processing that updates target systems at scheduled intervals (perhaps once a day or hour), CDC updates target systems instantly as changes occur. This means your analytics dashboard can reflect customer purchases within seconds rather than waiting for the next batch job.

**Exceptional Efficiency**: CDC processes only the data that has actually changed, dramatically reducing resource usage compared to full dataset transfers. Instead of copying an entire million-row table every hour, CDC might only transfer the dozen rows that actually changed.

**Guaranteed Consistency**: CDC ensures that downstream systems accurately reflect changes in the source database. When a customer updates their address in your main application, that change propagates reliably to your data warehouse, recommendation engine, and reporting systems.

## How Does CDC Work?

The magic of CDC lies in its ability to capture changes from a source database's write-ahead log (WAL), which is essentially the database's diary of all modifications. Every database maintains this log for recovery purposes, and CDC leverages this existing infrastructure.

Here's how the process flows: when you make a change to your PostgreSQL database, that change first gets written to the WAL before being applied to the actual data files. CDC tools read these WAL entries and convert them into events that can be streamed to target systems. For example, tools like Debezium read these logs and stream changes to Apache Kafka, which then delivers them to targets such as Cassandra.

## Debezium Connector (Source)

Debezium represents one of the most popular open-source platforms for CDC. It captures changes from various databases including PostgreSQL, MySQL, and Oracle, then streams these changes to Apache Kafka. Think of Debezium as a translator that speaks both "database language" and "streaming language."

### How Debezium Works

Debezium connectors act as careful monitors of a database's WAL, detecting every change that occurs. Each detected change gets converted into a structured event and sent to a designated Kafka topic. This process happens with remarkable precision:

When you insert a new row into a PostgreSQL table, Debezium generates an "INSERT" event containing all the new data. For updates, Debezium creates both "before" and "after" events, showing you exactly what changed from the old values to the new ones. Delete operations generate "DELETE" events that capture what was removed.

### Simple Example: Debezium with PostgreSQL

Let's walk through setting up Debezium with PostgreSQL using a practical example. Imagine you have a PostgreSQL table called `users` with columns for id, name, and email.

#### Step 1: Enable WAL in PostgreSQL

First, you need to configure PostgreSQL to use logical replication, which Debezium requires to read changes:

```bash
# Edit the PostgreSQL configuration file
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Update these settings in the configuration file:

```
wal_level = logical
max_wal_senders = 1
max_replication_slots = 1
```

After making these changes, restart PostgreSQL to apply them:

```bash
sudo systemctl restart postgresql
```

Next, grant the necessary replication permissions to your database user:

```bash
psql -U postgres -c "ALTER USER myuser WITH REPLICATION;"
```

#### Step 2: Set Up Kafka and Debezium

Now you'll set up the streaming infrastructure. Download and extract Apache Kafka:

```bash
wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0
```

Start Zookeeper and Kafka in separate terminals. Zookeeper manages Kafka's configuration, while Kafka handles the actual message streaming:

```bash
# Terminal 1: Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Terminal 2: Start Kafka
bin/kafka-server-start.sh config/server.properties
```

Download and set up the Debezium PostgreSQL connector:

```bash
mkdir -p /path/to/kafka/plugins
wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/2.7.0.Final/debezium-connector-postgres-2.7.0.Final-plugin.tar.gz
tar -xzf debezium-connector-postgres-2.7.0.Final-plugin.tar.gz -C /path/to/kafka/plugins
```

Configure Kafka Connect to recognize your Debezium plugin:

```bash
nano config/connect-distributed.properties
```

Add this line to tell Kafka Connect where to find plugins:

```
plugin.path=/path/to/kafka/plugins
```

Start Kafka Connect in distributed mode:

```bash
bin/connect-distributed.sh config/connect-distributed.properties
```

Create a configuration file for your Debezium connector. This JSON file tells Debezium exactly how to connect to your PostgreSQL database and which tables to monitor:

```json
{
  "name": "postgres-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "localhost",
    "database.port": "5432",
    "database.user": "myuser",
    "database.password": "mypassword",
    "database.dbname": "mydb",
    "database.server.name": "server1",
    "table.include.list": "public.users",
    "plugin.name": "pgoutput"
  }
}
```

Deploy the connector to start monitoring your database:

```bash
curl -X POST -H "Content-Type: application/json" --data @postgres-connector.json http://localhost:8083/connectors
```

#### Step 3: Observe Changes in Action

Now for the exciting part—watching CDC work in real-time. Create your users table and insert some data:

```sql
psql -U myuser -d mydb -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name TEXT, email TEXT);"
psql -U myuser -d mydb -c "INSERT INTO users (id, name, email) VALUES (1, 'Alice', 'alice@example.com');"
```

Debezium captures this insert operation and creates an event in a Kafka topic named `server1.public.users`. You can view this event using Kafka's console consumer:

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic server1.public.users --from-beginning
```

The event structure looks like this:

```json
{
  "schema": { ... },
  "payload": {
    "before": null,
    "after": { "id": 1, "name": "Alice", "email": "alice@example.com" },
    "op": "c", // 'c' indicates create (insert)
    "ts_ms": 1697051234567
  }
}
```

Notice how the event includes both "before" and "after" states. For an insert, "before" is null since the row didn't exist previously. The "op" field indicates the operation type, and "ts_ms" provides a timestamp.

## Cassandra Sink Connector

The Cassandra sink connector completes the CDC pipeline by reading data from Kafka topics and writing it to a Cassandra database. This connector excels at storing CDC events in a scalable, distributed NoSQL environment.

### How Cassandra Sink Connector Works

The connector acts as a bridge between Kafka and Cassandra, reading events from Kafka topics and translating them into appropriate Cassandra operations. It handles the mapping between Kafka record structures and Cassandra table schemas, managing inserts, updates, and deletes automatically.

### Simple Example: Cassandra Sink Connector

Let's continue our example by setting up Cassandra to receive the user events from our Debezium setup.

#### Step 1: Set Up Cassandra

Install and start Cassandra on Ubuntu:

```bash
sudo apt update
sudo apt install cassandra
sudo systemctl start cassandra
```

Verify that Cassandra is running properly:

```bash
nodetool status
```

Create a keyspace and table structure that matches your source data:

```bash
cqlsh -e "CREATE KEYSPACE mykeyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};"
cqlsh -e "CREATE TABLE mykeyspace.users (id int PRIMARY KEY, name text, email text);"
```

#### Step 2: Configure Cassandra Sink Connector

Download the Cassandra sink connector:

```bash
wget https://d1i4a8756m6x7j.cloudfront.net/repo/7.5/confluent-kafka-connect-cassandra-1.5.0.tar.gz
tar -xzf confluent-kafka-connect-cassandra-1.5.0.tar.gz -C /path/to/kafka/plugins
```

Create a configuration file that tells the connector how to map Kafka events to Cassandra records:

```json
{
  "name": "cassandra-sink",
  "config": {
    "connector.class": "io.confluent.connect.cassandra.CassandraSinkConnector",
    "tasks.max": "1",
    "topics": "server1.public.users",
    "cassandra.contact.points": "localhost",
    "cassandra.keyspace": "mykeyspace",
    "cassandra.table.name": "users",
    "cassandra.kcql": "INSERT INTO users SELECT id, name, email FROM server1.public.users"
  }
}
```

Deploy the connector to start the data flow:

```bash
curl -X POST -H "Content-Type: application/json" --data @cassandra-sink.json http://localhost:8083/connectors
```

#### Step 3: Observe Data in Cassandra

When Debezium sends user events to Kafka, the Cassandra sink connector automatically writes them to your Cassandra table. Query Cassandra to see the results:

```bash
cqlsh -e "SELECT * FROM mykeyspace.users;"
```

You should see the result: `id=1, name='Alice', email='alice@example.com'`.

The beauty of this setup is that any changes you make to the PostgreSQL users table will automatically appear in Cassandra within seconds, maintaining perfect synchronization between your systems.

## Key Considerations

When implementing CDC in production environments, several important factors require careful attention:

**Performance Impact**: CDC does add some overhead to your source database since it needs to read and process the WAL continuously. Monitor WAL usage in PostgreSQL, especially in high-transaction systems where the log can grow quickly. Consider the additional I/O load and ensure your database server has adequate resources.

**Schema Evolution**: Debezium handles schema changes gracefully—when you add columns to a PostgreSQL table, Debezium automatically detects and includes them in future events. However, you must ensure that your target systems (like Cassandra tables) can accommodate these schema changes. Plan your schema evolution strategy carefully.

**Scalability Considerations**: Cassandra's distributed architecture makes it excellent for handling high-volume CDC streams. Configure appropriate replication factors for reliability, and consider partitioning strategies that align with your query patterns and data access requirements.

## Assignment: Hands-On Project

To solidify your understanding of CDC concepts, I encourage you to work through a practical implementation project. This hands-on experience will help you see how all the pieces fit together in a real-world scenario.

**Project Repository**: [LuxDevHQ Data Engineering Project](https://github.com/LuxDevHQ/LuxDevHQ-Data-Engineering-Project)

**Your Task**: Clone the repository and follow the comprehensive instructions to set up a complete CDC pipeline using Debezium and a sink connector. The project guides you through using Linux commands to set up PostgreSQL tables, make various changes (inserts, updates, deletes), and verify that these changes propagate correctly to the target system.

```bash
git clone https://github.com/LuxDevHQ/LuxDevHQ-Data-Engineering-Project.git
cd LuxDevHQ-Data-Engineering-Project
```

Follow the project's README file for detailed setup and execution instructions. This project provides a practical environment where you can experiment with CDC concepts, helping you understand how Debezium and sink connectors work together in real-world scenarios.

As you work through this project, pay attention to how the different components interact, observe the timing of data propagation, and experiment with different types of database changes to see how they're handled by the CDC pipeline.
