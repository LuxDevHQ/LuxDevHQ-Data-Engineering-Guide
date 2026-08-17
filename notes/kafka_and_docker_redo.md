### **Docker, Docker Hub, Apache Kafka and Kafdrop — Beginner Class Handout**

### **Class Overview**

This week, we will learn how modern Data Engineering systems use **Docker**, **Docker Hub**, **Apache Kafka**, and **Kafdrop* following a practical approach.

We will **not start by running Kafka inside Docker**, instead, we will first install Kafka directly on Linux so that you understand what is happening behind the scenes.

**You should understand concepts such as:**
* Kafka brokers
* Kafka configuration
* Kafka ports
* Kafka logs
* KRaft
* Topics
* Producers
* Consumers
* Partitions
* Offsets
* Consumer groups
* Message retention

After understanding Kafka manually, we will learn how Docker can simplify the same setup. By the end of the training, we should be able to run something similar to:

```python
Python Producer
        |
        v
   Apache Kafka
        |
        v
   Kafka Topic
        |
        v
 Python Consumer
        |
        v
   PostgreSQL
```

We will also use **Kafdrop** to monitor Kafka. Later, the infrastructure will run using Docker:

```python
Docker Compose
|
|-- Kafka
|-- Kafdrop
|-- PostgreSQL
|-- Python Producer
|-- Python Consumer
```

---

### **Learning Objectives**

By the end of this training, you should be able to:

1. Explain what Docker is.
2. Explain the difference between a Docker image and a container.
3. Install Docker.
4. Pull Docker images.
5. Run Docker containers.
6. Stop and remove containers.
7. Understand Docker ports.
8. Understand Docker volumes.
9. Understand Docker networks.
10. Write a basic Dockerfile.
11. Build your own Docker image.
12. Create a Docker Hub account.
13. Tag Docker images.
14. Push an image to Docker Hub.
15. Pull your image from Docker Hub.
16. Understand Docker Compose.
17. Explain what Apache Kafka is.
18. Install Kafka manually.
19. Understand Kafka brokers.
20. Understand Kafka topics.
21. Understand Kafka partitions.
22. Create Kafka topics.
23. Produce messages into Kafka.
24. Consume messages from Kafka.
25. Understand Kafka offsets.
26. Understand Kafka consumer groups.
27. Understand Kafka message retention.
28. Understand Kafka KRaft.
29. Understand Kafka configuration files.
30. Understand Kafka ports and logs.
31. Explain what Kafdrop is.
32. Use Kafdrop to inspect Kafka topics.
33. Use Kafdrop to inspect messages.
34. Use Kafdrop to inspect consumer groups.
35. Run Kafka and Kafdrop using Docker.
36. Run Kafka and Kafdrop using Docker Compose.
37. Build a beginner-friendly real-time Data Engineering project.

---

### **Part 1: Introduction to Docker**

### **What is Docker?**

Docker is a platform that allows us to package applications together with everything they need to run.

This can include:

* Application code
* Programming language runtime
* Libraries
* Dependencies
* Configuration
* Environment settings

Instead of saying:

> It works on my machine.

Docker helps us create an environment that can behave consistently across different machines. For example, imagine that you create a Python application.

Your application may require:

```python
Python 3.x
FastAPI
Requests
Pandas
PostgreSQL Driver
Environment Variables
```

Another student may have a different Python version or may be missing some libraries.

Docker allows us to package these requirements together.

Conceptually:

```text
Application Code
      +
Dependencies
      +
Runtime
      +
Configuration
      |
      v
Docker Image
```

The Docker image can then be used to create containers.

---

### **Why Do Data Engineers Use Docker?**

Data Engineering projects normally involve several technologies.

For example:

```text
Python
Kafka
PostgreSQL
Airflow
Spark
Grafana
Prometheus
Redis
dbt
```

Installing every tool directly on your computer can become difficult.

Different tools may require different:

* Versions
* Libraries
* Ports
* Configurations
* Operating system packages

Docker allows us to isolate these applications.

For example:

```text
Docker
|
|-- Kafka Container
|-- PostgreSQL Container
|-- Grafana Container
|-- Python Container
```

Each application can run inside its own container.

---

### **Docker Image vs Docker Container**

This is one of the most important Docker concepts.

A **Docker image** is a packaged template.

A **Docker container** is a running instance of that image.

Think about it like this:

```text
Docker Image
    |
    | docker run
    v
Docker Container
```

Another simple analogy:

```text
Class        -> Object
Blueprint    -> Building
Image        -> Container
```

The image contains the instructions.

The container is the running application.

---

### **Example**

Suppose we pull the Nginx image:

```bash
docker pull nginx
```

The image is stored locally.

We can then create a container:

```bash
docker run nginx
```

The running instance is the container.

---

### **Checking Docker**

After installing Docker, confirm that it is available:

```bash
docker --version
```

You can also run:

```bash
docker info
```

This provides more information about your Docker environment.

---

### **Important Docker Commands**

Check currently running containers:

```bash
docker ps
```

Check all containers, including stopped containers:

```bash
docker ps -a
```

View images:

```bash
docker images
```

Pull an image:

```bash
docker pull nginx
```

Run a container:

```bash
docker run nginx
```

Run a container in the background:

```bash
docker run -d nginx
```

The `-d` means:

```text
Detached Mode
```

The container runs in the background.

---

### **Container Names**

Instead of allowing Docker to generate a random container name, we can specify one:

```bash
docker run -d --name my-nginx nginx
```

Now check:

```bash
docker ps
```

You should see:

```text
my-nginx
```

---

### **Stopping Containers**

Stop a container:

```bash
docker stop my-nginx
```

Start it again:

```bash
docker start my-nginx
```

Restart:

```bash
docker restart my-nginx
```

---

### **Removing Containers**

Before removing a running container, stop it:

```bash
docker stop my-nginx
```

Then remove it:

```bash
docker rm my-nginx
```

Force removal:

```bash
docker rm -f my-nginx
```

---

### **Removing Docker Images**

View images:

```bash
docker images
```

Remove an image:

```bash
docker rmi nginx
```

---

### **Docker Logs**

Containers run applications, and those applications generate logs.

View logs:

```bash
docker logs my-nginx
```

Follow logs continuously:

```bash
docker logs -f my-nginx
```

This is very important in Data Engineering.

For example:

```text
Kafka is not starting
        |
        v
Check Kafka container logs
```

```bash
docker logs kafka
```

Logs are normally one of the first places we investigate when troubleshooting.

---

### **Executing Commands Inside a Container**

You can enter a running container.

Example:

```bash
docker exec -it my-container bash
```

Sometimes the image may only contain `sh`:

```bash
docker exec -it my-container sh
```

This allows you to investigate the environment inside the container.

---

### **Docker Ports**

Applications usually listen on network ports.

Examples:

```text
Application       Common Port

PostgreSQL        5432
Kafka             9092
Grafana           3000
Kafdrop            9000
FastAPI           8000
```

Imagine an application inside a container listens on:

```text
Port 80
```

You can expose it to your computer using:

```bash
docker run -p 8080:80 nginx
```

Read this as:

```text
HOST:CONTAINER

8080:80
```

Meaning:

```text
Your Laptop Port 8080
        |
        v
Container Port 80
```

You can then access:

```text
http://localhost:8080
```

---

### **Docker Volumes**

Containers are designed to be replaceable.

This means data stored only inside a container may disappear when the container is removed.

Imagine PostgreSQL running in a container.

Without persistent storage:

```text
PostgreSQL Container
       |
       v
Database Data
```

If the container is removed, you may lose the database data.

Docker volumes solve this problem.

Conceptually:

```text
Container
    |
    v
Docker Volume
    |
    v
Persistent Data
```

Create a volume:

```bash
docker volume create postgres_data
```

List volumes:

```bash
docker volume ls
```

Volumes are especially important for:

* PostgreSQL
* Kafka
* Grafana
* Databases
* Persistent application data

---

### **Docker Networks**

Containers often need to communicate with other containers.

Imagine:

```text
Producer Container
       |
       v
Kafka Container
       |
       v
Consumer Container
```

Docker networks allow containers to communicate.

Create a network:

```bash
docker network create data-network
```

List networks:

```bash
docker network ls
```

A major advantage of Docker networking is that containers can normally communicate using container or service names.

Instead of:

```text
localhost
```

we may use:

```text
kafka
```

or:

```text
postgres
```

depending on our Docker configuration.

---

### **Part 2: Dockerfile**

### **What is a Dockerfile?**

A Dockerfile is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

---

### **Understanding the Dockerfile**

#### **FROM**

```dockerfile
FROM python:3.12-slim
```

This defines the base image.

We are saying:

> Start with an environment that already has Python installed.

---

#### **WORKDIR**

```dockerfile
WORKDIR /app
```

This sets the working directory inside the image.

---

#### **COPY**

```dockerfile
COPY . .
```

This copies files from your machine into the Docker image.

---

#### **RUN**

```dockerfile
RUN pip install -r requirements.txt
```

This executes a command while building the image.

In this example, Docker installs Python libraries.

---

#### **CMD**

```dockerfile
CMD ["python", "app.py"]
```

This defines what should run when the container starts.

---

### **Building Your First Docker Image**

Suppose your project looks like:

```text
my-project/
|
|-- app.py
|-- requirements.txt
|-- Dockerfile
```

Build the image:

```bash
docker build -t my-python-app .
```

Check the image:

```bash
docker images
```

Run it:

```bash
docker run my-python-app
```

---

### **Docker Image Tags**

Docker images can have versions.

Example:

```text
my-python-app:v1
my-python-app:v2
my-python-app:latest
```

Build with a tag:

```bash
docker build -t my-python-app:v1 .
```

Tags help us identify different versions of an application.

---

### **Part 3: Docker Hub**

### **What is Docker Hub?**

Docker Hub is an online registry where Docker images can be stored and shared.

Think of it as a central location for Docker images.

A simple beginner analogy is:

```text
GitHub      -> Source Code
Docker Hub  -> Docker Images
```

This is only an analogy, but it helps explain their roles.

---

### **Why Publish Docker Images?**

Imagine you build an application on your laptop.

You want another student to run it.

Without Docker, they may need:

```text
Install Python
Install libraries
Configure environment
Install dependencies
Resolve version problems
```

With Docker:

```text
docker pull username/application:v1
```

Then:

```bash
docker run username/application:v1
```

---

### **Logging Into Docker Hub**

From the terminal:

```bash
docker login
```

Enter your Docker Hub credentials when requested.

---

### **Tagging an Image**

Suppose your local image is:

```text
my-python-app:v1
```

Your Docker Hub username is:

```text
student123
```

Tag it:

```bash
docker tag my-python-app:v1 student123/my-python-app:v1
```

Check:

```bash
docker images
```

You should now see:

```text
student123/my-python-app:v1
```

---

### **Pushing to Docker Hub**

Push the image:

```bash
docker push student123/my-python-app:v1
```

The image should now be available from your Docker Hub repository.

---

### **Pulling the Image**

On another machine:

```bash
docker pull student123/my-python-app:v1
```

Run it:

```bash
docker run student123/my-python-app:v1
```

This demonstrates one of the biggest benefits of containerization.

---

### **Docker Hub Exercise**

Each student should:

1. Create a simple Python application.
2. Create a Dockerfile.
3. Build the Docker image.
4. Run the Docker container locally.
5. Create a Docker Hub repository.
6. Tag the image.
7. Push the image.
8. Delete the image locally.
9. Pull it again from Docker Hub.
10. Run the downloaded image.

The objective is to understand this lifecycle:

```text
Code
 |
 v
Dockerfile
 |
 v
Docker Build
 |
 v
Docker Image
 |
 v
Docker Hub
 |
 v
Docker Pull
 |
 v
Container
```

---

### **Part 4: Introduction to Apache Kafka**

### **What is Apache Kafka?**

Apache Kafka is a distributed event-streaming platform.

It allows applications to publish and consume streams of data.

Kafka is commonly used when data needs to move between systems continuously.

Examples:

* Bank transactions
* Mobile money transactions
* Website activity
* Application logs
* IoT sensors
* GPS data
* Stock market data
* Cryptocurrency prices
* E-commerce orders
* Fraud detection events
* System monitoring events

---

### **Batch Processing vs Streaming**

Suppose a company processes transactions once every day.

That is closer to:

```text
Transactions
     |
     v
Wait Until End of Day
     |
     v
Process Everything
```

This is batch processing.

Streaming works differently:

```text
Transaction 1 -> Process
Transaction 2 -> Process
Transaction 3 -> Process
Transaction 4 -> Process
```

Data is processed continuously as it arrives.

Kafka helps us build this type of architecture.

---

### **Simple Kafka Architecture**

```text
Producer
    |
    v
Kafka
    |
    v
Topic
    |
    v
Consumer
```

---

### **Kafka Producer**

A producer sends messages or events into Kafka.

Examples of producers:

* Python application
* Java application
* Website backend
* Payment system
* Mobile application
* IoT sensor

Example:

```text
Python Application
       |
       v
Kafka Producer
       |
       v
Kafka Topic
```

---

### **Kafka Consumer**

A consumer reads messages from Kafka.

Example:

```text
Kafka Topic
     |
     v
Python Consumer
     |
     v
PostgreSQL
```

---

### **Kafka Topic**

A Kafka topic is a named stream where messages are stored.

Examples:

```text
transactions
customer_events
website_clicks
eth_prices
orders
payments
fraud_alerts
```

Think of a topic as a category or channel for related events.

For example:

```text
Topic: payments
```

might contain:

```json
{
  "transaction_id": 1001,
  "customer": "C001",
  "amount": 5000
}
```

and:

```json
{
  "transaction_id": 1002,
  "customer": "C002",
  "amount": 2500
}
```

---

### **Kafka Broker**

A broker is a Kafka server.

It receives, stores, and serves Kafka messages.

Simple setup:

```text
Producer
   |
   v
Kafka Broker
   |
   v
Consumer
```

Larger Kafka environments may contain multiple brokers:

```text
Kafka Cluster
|
|-- Broker 1
|-- Broker 2
|-- Broker 3
```

Together, these brokers form a Kafka cluster.

---

### **Why Multiple Brokers?**

Multiple brokers can provide:

* Scalability
* Availability
* Fault tolerance
* Distribution of data
* Higher throughput

For beginner training, we can start with one broker before moving to multiple brokers.

---

### **Kafka Ports**

Kafka brokers communicate over network ports.

A commonly encountered Kafka listener port is:

```text
9092
```

When troubleshooting Kafka, always ask:

```text
Is Kafka running?
Which port is Kafka listening on?
Can my producer reach that port?
Can my consumer reach that port?
```

Port problems are common when:

* Using Docker
* Using cloud servers
* Using firewalls
* Using virtual machines
* Using multiple Kafka listeners

---

### **Kafka Configuration**

Kafka behaviour is controlled through configuration.

Examples of things we can configure include:

* Broker listeners
* Ports
* Log directories
* Message retention
* Topic defaults
* Replication
* KRaft settings
* Broker IDs
* Controller configuration

Students should understand that Kafka does not simply "start magically."

It reads configuration that determines how the broker behaves.

---

### **Kafka Logs**

Kafka writes logs that help us understand what the server is doing.

When Kafka fails, do not immediately reinstall it.

Check:

```text
Kafka logs
Configuration
Ports
Permissions
Storage directories
Java
Network
```

Logs help answer questions such as:

```text
Why did Kafka fail to start?

Is another service using the port?

Can Kafka access its data directory?

Did KRaft initialize correctly?
```

---

### **What is KRaft?**

KRaft is Kafka's built-in metadata management system.

Kafka needs to keep information about:

* Brokers
* Topics
* Partitions
* Cluster metadata
* Leadership
* Configuration

KRaft allows Kafka to manage this metadata without requiring a separate ZooKeeper service.

Conceptually:

```text
Older Architecture

Kafka
  |
  v
ZooKeeper
```

Modern Kafka architecture:

```text
Kafka
  |
  v
KRaft
```

For our training, we will focus on understanding Kafka using KRaft.

---

### **Why Learn Kafka Without Docker First?**

Running:

```bash
docker compose up -d
```

can make Kafka appear easy.

But if you do not understand Kafka itself, troubleshooting becomes difficult.

We first want to understand:

```text
Where Kafka is installed
How Kafka starts
Which configuration it reads
Which ports it uses
Where logs are stored
How KRaft works
How topics are created
How producers connect
How consumers connect
```

After understanding these concepts, Docker becomes much easier.

---

### **Part 5: Kafka Topics**

A topic is where events are published.

Example:

```text
Topic: customer_transactions
```

Messages:

```text
Transaction A
Transaction B
Transaction C
Transaction D
```

---

### **Partitions**

Kafka topics are divided into partitions.

Example:

```text
customer_transactions
|
|-- Partition 0
|-- Partition 1
|-- Partition 2
```

Partitions allow Kafka to distribute data and process messages at scale.

---

### **Offsets**

Inside each partition, Kafka messages have positions called offsets.

Example:

```text
Partition 0

Offset 0 -> Transaction A
Offset 1 -> Transaction B
Offset 2 -> Transaction C
Offset 3 -> Transaction D
```

The offset tells us the position of a message inside a partition.

---

### **Why Are Offsets Important?**

Imagine a consumer reads:

```text
Offset 0
Offset 1
Offset 2
```

Then the application stops.

When it restarts, Kafka can help the consumer continue from where it left off depending on its configuration and committed offsets.

Conceptually:

```text
Consumer Last Position
        |
        v
     Offset 2
        |
        v
Continue from later messages
```

---

### **Producer and Consumer Example**

```text
Python Producer
      |
      | Sends:
      |
      | Transaction A
      | Transaction B
      | Transaction C
      v
Kafka Topic
      |
      v
Python Consumer
      |
      v
PostgreSQL
```

---

### **Consumer Groups**

Consumers can work together in a consumer group.

Example:

```text
                  Kafka Topic
                      |
          -------------------------
          |                       |
          v                       v
     Consumer 1              Consumer 2
          |                       |
          -------------------------
                    |
                    v
              Consumer Group
```

Consumer groups allow Kafka workloads to be distributed across multiple consumers.

---

### **Consumer Lag**

Consumer lag shows how far behind a consumer is compared with the latest messages.

Example:

Kafka has reached:

```text
Offset 1000
```

Consumer has processed up to:

```text
Offset 800
```

The consumer is behind by approximately:

```text
200 messages
```

This is called consumer lag.

Large lag may indicate:

* Slow consumer
* Failed consumer
* Heavy workload
* Network problem
* Database bottleneck
* Processing bottleneck

Monitoring consumer lag is important in production Kafka systems.

---

### **Kafka Message Retention**

Kafka does not normally delete a message simply because a consumer has read it.

Example:

```text
Producer
   |
   v
Kafka Topic
   |
   v
Consumer Reads Message
   |
   v
Message Can Still Remain in Kafka
```

Kafka removes old data according to its retention settings.

---

### **Retention Time**

Retention time defines approximately how long Kafka should keep messages before they become eligible for deletion.

A common property is:

```properties
retention.ms
```

Example:

```properties
retention.ms=86400000
```

This represents approximately one day.

---

### **Retention by Size**

Kafka can also limit how much data is retained.

A related configuration is:

```properties
retention.bytes
```

This allows retention policies to consider storage size.

---

### **Important Retention Concept**

Remember:

```text
Consumer Reads Message
        !=
Kafka Immediately Deletes Message
```

Kafka can retain data even after consumers have processed it.

This makes Kafka useful for:

* Reprocessing
* Multiple consumers
* Debugging
* Recovering systems
* Event-driven architectures

---

### **Part 6: Kafka CLI Practice**

Students should learn Kafka from the command line before relying on graphical tools.

Typical activities include:

1. Start Kafka.
2. Check Kafka logs.
3. Create a topic.
4. List topics.
5. Describe a topic.
6. Start a console producer.
7. Send messages.
8. Start a console consumer.
9. Read messages.
10. Explore offsets.
11. Explore consumer groups.

The exact command paths can vary depending on your Kafka installation and operating system, so focus on understanding what each command is doing.

---

### **Example Topic**

Create a topic conceptually called:

```text
student-events
```

Produce messages:

```text
Harun started Docker
Mary completed Python
John started Kafka
Alice completed SQL
```

Consume those messages from another terminal.

This simple exercise helps students understand:

```text
Producer
   |
   v
Kafka Topic
   |
   v
Consumer
```

---

### **Part 7: Introduction to Kafdrop**

### **What is Kafdrop?**

Kafdrop is a web-based user interface used to inspect and monitor Kafka.

Kafka itself does not depend on Kafdrop.

Kafdrop is simply a tool that makes it easier for us to see what is happening inside Kafka.

---

### **Why Use Kafdrop?**

Without Kafdrop, many Kafka tasks are performed using command-line tools.

Kafdrop gives us a visual interface.

We can use it to inspect:

* Kafka brokers
* Topics
* Partitions
* Messages
* Consumer groups
* Offsets
* Consumer lag

---

### **Kafka and Kafdrop Architecture**

```text
Producer
    |
    v
Apache Kafka
    |
    v
Topic
    |
    v
Consumer

Kafdrop
    |
    v
Monitors Kafka
```

Another way to visualize it:

```text
               Kafdrop
                  |
                  v
Producer ---> Kafka ---> Consumer
                  |
                  v
                Topic
```

Kafdrop is not the producer.

Kafdrop is not the consumer.

Kafdrop is not Kafka.

It is a tool used to inspect Kafka.

---

### **What Should You Check in Kafdrop?**

When opening Kafdrop, try to identify:

#### Brokers

```text
How many Kafka brokers are running?
```

#### Topics

```text
Which topics exist?
```

#### Partitions

```text
How many partitions does the topic have?
```

#### Messages

```text
Are messages reaching Kafka?
```

#### Consumer Groups

```text
Which applications are consuming data?
```

#### Consumer Lag

```text
Are consumers keeping up with producers?
```

---

### **Kafdrop Exercise**

1. Start Kafka.
2. Start Kafdrop.
3. Open Kafdrop in the browser.
4. Create a Kafka topic.
5. Refresh Kafdrop.
6. Find the topic.
7. Start a producer.
8. Send JSON messages.
9. Inspect the messages in Kafdrop.
10. Start a consumer.
11. Observe the consumer group.
12. Discuss consumer lag.

---

### **Part 8: Moving Back to Docker**

Once we understand Kafka manually, we return to Docker.

At this stage, when we see:

```text
Kafka Docker Container
```

we already understand what is inside.

Docker is not replacing our Kafka knowledge.

Docker is simplifying deployment.

---

### **Manual Kafka vs Docker Kafka**

Without Docker:

```text
Linux Server
|
|-- Java
|-- Kafka Installation
|-- Kafka Config
|-- Kafka Logs
|-- Kafka Data
|-- Kafka Process
```

With Docker:

```text
Linux Server
|
|-- Docker
    |
    |-- Kafka Container
```

The Kafka broker still needs:

* Configuration
* Ports
* Storage
* Logs
* Networking

Docker simply changes how we package and run it.

---

### **Running Kafka with Docker**

Conceptually:

```text
Kafka Docker Image
       |
       v
docker run
       |
       v
Kafka Container
       |
       v
Kafka Broker Running
```

We then need to understand:

* Kafka listeners
* Container ports
* Host ports
* Docker networks
* Environment variables
* Volumes

---

### **Why Docker Networking Matters for Kafka**

Suppose we have:

```text
Producer Container
Kafka Container
Consumer Container
```

They need to communicate.

Docker provides networking:

```text
Docker Network
|
|-- Producer
|-- Kafka
|-- Consumer
```

Applications can then communicate using service names depending on configuration.

Example:

```text
Producer
   |
   v
kafka:9092
```

rather than always using:

```text
localhost:9092
```

This distinction becomes very important when working with Kafka in containers.

---

### **Part 9: Docker Compose**

### **What is Docker Compose?**

Docker Compose allows us to define and run multiple containers together.

Imagine running:

```text
Kafka
Kafdrop
PostgreSQL
Producer
Consumer
```

Running all of these manually using individual `docker run` commands can become difficult.

Docker Compose allows us to define the infrastructure in one configuration file.

---

### **Conceptual Docker Compose File**

```yaml
services:

  kafka:
    # Kafka configuration

  kafdrop:
    # Kafdrop configuration

  postgres:
    # PostgreSQL configuration

  producer:
    # Python producer

  consumer:
    # Python consumer
```

Then we can start the environment using:

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

Stop everything:

```bash
docker compose down
```

---

### **Why Docker Compose is Important in Data Engineering**

Consider this architecture:

```text
API
 |
 v
Python Producer
 |
 v
Kafka
 |
 v
Python Consumer
 |
 v
PostgreSQL
 |
 v
Grafana
```

Using Docker Compose we can manage these services together:

```text
Docker Compose
|
|-- Kafka
|-- Kafdrop
|-- PostgreSQL
|-- Grafana
|-- Producer
|-- Consumer
```

This makes development environments easier to:

* Start
* Stop
* Share
* Reproduce
* Debug
* Deploy

---

### **Part 10: Kafka and Kafdrop with Docker Compose**

After learning both technologies separately, we combine them.

Architecture:

```text
                   Docker Network
                         |
      -----------------------------------------
      |                  |                    |
      v                  v                    v
   Producer            Kafka              Kafdrop
                          |
                          v
                       Consumer
```

Kafdrop connects to Kafka and allows us to inspect the broker and topics.

---

### **Part 11: Beginner Kafka Project**

### **Project: Student Activity Event Streaming**

We can create a simple system that streams student activity.

Producer generates events such as:

```json
{
  "student_id": 101,
  "student_name": "Mary",
  "course": "Data Engineering",
  "activity": "Completed Kafka Lesson"
}
```

Another event:

```json
{
  "student_id": 102,
  "student_name": "John",
  "course": "Data Engineering",
  "activity": "Started Docker Lesson"
}
```

---

### **Project Architecture**

```text
Python Producer
      |
      v
Kafka Topic
student-events
      |
      v
Python Consumer
      |
      v
PostgreSQL
```

Kafdrop:

```text
                  Kafdrop
                     |
                     v
Python Producer -> Kafka -> Python Consumer
```

---

### **What Students Should Practice**

#### Producer

Write a Python application that sends JSON data.

#### Topic

Create:

```text
student-events
```

#### Consumer

Write a Python application that reads the messages.

#### Kafdrop

Confirm that messages are reaching Kafka.

#### PostgreSQL

Save consumed events into a database.

---

### **Part 12: Real-Time API Project**

Once the beginner project works, replace manually generated events with API data.

Architecture:

```text
Public API
    |
    v
Python Producer
    |
    v
Kafka
    |
    v
Kafka Topic
    |
    v
Python Consumer
    |
    v
PostgreSQL
```

Kafdrop monitors the Kafka layer.

---

### **Possible Data Sources**

Examples include:

* Weather data
* Cryptocurrency prices
* Public transport data
* News APIs
* E-commerce simulation
* Application logs
* IoT simulation
* Banking transaction simulation

---

### **Part 13: Full Data Engineering Architecture**

Later, our architecture could become:

```text
                         Kafdrop
                            |
                            v
API ---> Python ---> Apache Kafka ---> Consumer
          Producer        |                |
                          |                v
                          |           PostgreSQL
                          |
                          v
                     More Consumers
```

We can eventually add:

```text
Airflow
Spark
Grafana
Prometheus
Cloud Storage
Data Warehouse
```

Example:

```text
API
 |
 v
Python Producer
 |
 v
Kafka
 |
 v
Consumer
 |
 v
PostgreSQL
 |
 v
Grafana
```

---

### **Part 14: Troubleshooting Skills**

A Data Engineer should not only know how to install tools.

You should learn how to investigate when something fails.

---

### **When Docker Fails**

Check:

```bash
docker ps
```

Then:

```bash
docker ps -a
```

Then:

```bash
docker logs <container-name>
```

Check:

* Container status
* Port mapping
* Environment variables
* Volumes
* Networks
* Image
* Application logs

---

### **When Kafka Fails**

Check:

1. Is Java installed?
2. Is Kafka running?
3. Is the broker port open?
4. Is another process using the port?
5. Is KRaft initialized?
6. Is the Kafka configuration correct?
7. Can Kafka access its storage directory?
8. What do the Kafka logs say?

---

### **When the Producer Cannot Connect**

Check:

```text
Producer
    |
    X
    |
Kafka
```

Investigate:

* Kafka hostname
* Kafka port
* Kafka listener configuration
* Firewall
* Docker network
* Container names
* Broker status

---

### **When Kafdrop Cannot Connect**

Check:

```text
Kafdrop
   |
   X
   |
Kafka
```

Investigate:

* Kafka broker address
* Kafka port
* Docker network
* Kafka listeners
* Kafdrop configuration
* Kafka availability

---

### **When Consumer Lag Increases**

Architecture:

```text
Producer
  |
  v
Kafka
  |
  v
Consumer
```

If producers generate:

```text
1000 messages/minute
```

but consumers process:

```text
500 messages/minute
```

lag will increase.

Possible solutions include:

* Optimize consumer code
* Add consumers
* Increase partitions where appropriate
* Improve database performance
* Reduce processing bottlenecks

---

### **Part 15: Important Terms to Remember**

### **Docker**

Platform for building and running containers.

### **Docker Image**

Template used to create containers.

### **Container**

Running instance of an image.

### **Dockerfile**

Instructions used to build a Docker image.

### **Docker Hub**

Registry used to store and distribute Docker images.

### **Docker Volume**

Persistent storage used by containers.

### **Docker Network**

Allows containers to communicate.

### **Docker Compose**

Tool for defining and running multiple containers together.

### **Apache Kafka**

Distributed event-streaming platform.

### **Broker**

Kafka server.

### **Topic**

Named stream where Kafka messages are stored.

### **Producer**

Application that sends events to Kafka.

### **Consumer**

Application that reads events from Kafka.

### **Partition**

Subdivision of a Kafka topic.

### **Offset**

Position of a message within a partition.

### **Consumer Group**

Group of consumers working together.

### **Consumer Lag**

Difference between how far Kafka has progressed and how far the consumer has processed.

### **Retention**

Rules controlling how long or how much data Kafka keeps.

### **KRaft**

Kafka's built-in metadata management mechanism.

### **Kafdrop**

Web interface used to inspect and monitor Kafka.

---

### **Part 16: Commands You Should Be Comfortable With**

By the end of the Docker section:

```bash
docker --version
docker info
docker pull
docker images
docker run
docker ps
docker ps -a
docker stop
docker start
docker restart
docker logs
docker exec
docker rm
docker rmi
docker build
docker tag
docker login
docker push
docker volume ls
docker network ls
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

Do not only memorize these commands.

Understand **why** you are using each one.

---

### **Part 17: Recommended Learning Order**

We will follow this sequence:

```text
1. Understand Docker
        |
        v
2. Run Containers
        |
        v
3. Understand Images
        |
        v
4. Write Dockerfiles
        |
        v
5. Build Images
        |
        v
6. Publish to Docker Hub
        |
        v
7. Install Kafka Manually
        |
        v
8. Understand Kafka Architecture
        |
        v
9. Produce and Consume Messages
        |
        v
10. Understand Topics and Partitions
        |
        v
11. Understand Offsets and Consumer Groups
        |
        v
12. Understand Retention
        |
        v
13. Learn Kafdrop
        |
        v
14. Monitor Kafka
        |
        v
15. Learn Docker Compose
        |
        v
16. Run Kafka with Docker
        |
        v
17. Run Kafdrop with Docker
        |
        v
18. Build Real Data Engineering Projects
```

---

### **Part 18: Important Learning Principle**

Do not learn tools by only copying commands.

For every command you run, ask yourself:

```text
What am I starting?

Which port is it using?

Where is its configuration?

Where are its logs?

Where is its data stored?

Which application connects to it?

What happens if it fails?

How can I verify that it is running?
```

For Kafka, you should be able to explain:

```text
Producer
    |
    v
Broker
    |
    v
Topic
    |
    v
Partition
    |
    v
Offset
    |
    v
Consumer
```

For Docker, you should be able to explain:

```text
Source Code
    |
    v
Dockerfile
    |
    v
Docker Build
    |
    v
Docker Image
    |
    v
Docker Run
    |
    v
Container
```

For Docker Hub:

```text
Docker Image
    |
    v
Tag
    |
    v
Push
    |
    v
Docker Hub
    |
    v
Pull
    |
    v
Run Anywhere
```

---

### **Part 19: End-of-Class Practical Challenge**

Each student should be able to complete the following without simply copying a completed project.

#### **Challenge 1**

Install and verify Docker.

#### **Challenge 2**

Run a basic container.

#### **Challenge 3**

Create a Python application.

#### **Challenge 4**

Write a Dockerfile.

#### **Challenge 5**

Build your Python Docker image.

#### **Challenge 6**

Run the image as a container.

#### **Challenge 7**

Publish the image to Docker Hub.

#### **Challenge 8**

Pull the image onto another machine or after deleting the local copy.

#### **Challenge 9**

Install Kafka manually.

#### **Challenge 10**

Start the Kafka broker.

#### **Challenge 11**

Create a topic.

#### **Challenge 12**

Produce messages.

#### **Challenge 13**

Consume messages.

#### **Challenge 14**

Explain partitions and offsets.

#### **Challenge 15**

Explain message retention.

#### **Challenge 16**

Install and run Kafdrop.

#### **Challenge 17**

Use Kafdrop to inspect the Kafka topic.

#### **Challenge 18**

Run Kafka and Kafdrop using Docker.

#### **Challenge 19**

Convert the setup to Docker Compose.

#### **Challenge 20**

Build a Python producer and consumer around the Kafka infrastructure.

---

### **Final Takeaway**

The objective of this class is not simply to know commands such as:

```bash
docker compose up -d
```

The objective is to understand **what is actually being started**.

When Docker starts Kafka, you should already understand:

```text
What is the broker?

What is KRaft?

Which port is Kafka using?

Where are the logs?

Where is Kafka storing its data?

What is a topic?

What is a partition?

What is an offset?

How does a producer connect?

How does a consumer connect?

What is consumer lag?

Why is Kafdrop useful?
```

Once you understand those concepts, Docker becomes a tool for **simplifying deployment**, rather than a tool that hides technologies you do not understand.

Our overall journey is:

```text
Understand
    |
    v
Install
    |
    v
Configure
    |
    v
Use
    |
    v
Break
    |
    v
Troubleshoot
    |
    v
Containerize
    |
    v
Automate
    |
    v
Build Real Projects
```

**The goal is not just to run Kafka. The goal is to understand Kafka, understand Docker, and then use both technologies confidently to build real Data Engineering systems.**
