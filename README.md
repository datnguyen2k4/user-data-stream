# User Data Stream
## Overview
The User Data Stream project is a real-time data processing system designed to collect user data from a public API (https://randomuser.me/api/), stream it through Kafka, process it with Spark, store it in Cassandra, and visualize it using Superset. The entire system is containerized using Docker for ease of deployment and management.

## System Architecture
The system consists of the following key components, integrated to form a complete data processing pipeline:

*Apache Airflow*:

-Role: Workflow scheduling and management.

-Function: Fetches user data from the API, formats it, and sends it to the Kafka topic users_created.

*Apache Kafka*:

-Role: Message broker.

-Function: Streams data from Airflow to Spark using a publish-subscribe model.

*Apache Spark*:

-Role: Streaming data processing.

-Function: Reads data from the Kafka topic users_created, processes it, and writes it to Cassandra.

*Apache Cassandra*:

-Role: NoSQL database.

-Function: Stores user data in the created_users table within the spark_streams keyspace.

*Apache Superset*:

-Role: Data visualization.

-Function: Connects to Cassandra or Trino to create dashboards and charts.

*Trino*:

-Role: Distributed SQL query engine.

-Function: Queries data from Cassandra to support analysis and visualization.

*Docker*:

-Role: Containerization.

-Function: Packages and deploys all services in the system.

## Workflow

*Data Collection*: Airflow runs the user_automation DAG, fetching data from https://randomuser.me/api/ every minute, formatting it, and sending it to the Kafka topic users_created.

*Data Streaming*: Kafka acts as an intermediary, storing and forwarding data from Airflow to Spark.

*Data Processing*: Spark reads data from the users_created topic, processes it in a streaming fashion, and writes it to the created_users table in Cassandra.

*Data Storage*: Cassandra stores user data in a table with fields such as id, first_name, last_name, etc.

*Querying and Visualization*: Superset or Trino is used to query data from Cassandra and create visualizations or dashboards.

## Technologies Used
*Apache Airflow*: Workflow management and scheduling.

*Apache Kafka*: Real-time data streaming.

*Apache Spark*: Streaming data processing.

*Apache Cassandra*: NoSQL data storage.

*Apache Superset*: Data visualization.

*Trino*: Distributed SQL querying.

*Docker*: Containerization of services

## Installation and Usage Guide
### Prerequisites
Docker: Installed on the host machine.

Docker Compose: Installed to manage Docker services.
### Installation
#### Step 1: Clone the Repository: Clone the project source code from the repository (assuming you have a repository containing kafka_stream.py, spark_stream.py, and docker-compose.yml):

**git clone https://github.com/datnguyen2k4/user-data-stream.git**

**cd user_data_stream**

#### Step 2: Start the System: Use Docker Compose to launch all services:

**docker-compose up -d**

This command starts services like Zookeeper, Kafka, Airflow, Spark, Cassandra, Superset, Trino, etc., in the background.

#### Step 3: Activate the DAG in Airflow:
-Access the Airflow web interface at http://localhost:8080.

-Log in with the default credentials (if unchanged, configure username/password in docker-compose.yml).

-Locate the user_automation DAG in the list, toggle it "On," and manually trigger it if needed by clicking "Trigger DAG."

-The DAG runs every minute for 1 minute (as per the logic in kafka_stream.py).

#### Step 4: Submit the Spark Job:

-Access the Spark web interface at http://localhost:9090.

**docker exec -it user-data-stream-spark-worker-1 bash**

**spark-submit --master spark://spark-master:7077 --packages com.datastax.spark:spark-cassandra-connector_2.12:3.4.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.1,org.apache.kafka:kafka-clients:3.4.1  /opt/spark-apps/spark_stream.py**

#### Step 5: Check Data in Cassandra:

**docker exec -it cassandra cqlsh -u cassandra -p cassandra**

Run a query to verify the data:

**USE spark_streams**;

**SELECT * FROM created_users LIMIT 10**;

#### Step 6: Visualize Data with Superset:
-Access Superset at http://localhost:8088.

-Log in with the default admin credentials (username: admin, password: admin).

-Configure a connection to Cassandra or Trino:

-Go to "Settings" > "Database Connections" > "Add a new database."

-Select the driver: Trino and enter trino://trino@trino:8080/cassandra/spark_streams.

-Create dashboards or charts based on the data in the created_users table.