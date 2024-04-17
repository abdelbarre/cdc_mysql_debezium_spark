# Change Data Capture (CDC) with Docker, Apache Kafka, Debezium, and Apache Spark Streaming

## Introduction
Change Data Capture (CDC) plays a vital role in data engineering by enabling real-time data integration and analysis. This article presents a comprehensive guide to setting up a CDC pipeline using Docker Compose, Apache Kafka, Debezium, and Apache Spark Streaming. The architecture involves capturing changes from a MySQL database, processing them with Debezium, publishing to Kafka, and subsequently analyzing the data with Spark Streaming.

## Components Overview
- **MySQL**: The source database where changes are tracked.
- **Debezium**: Responsible for capturing changes from MySQL and publishing them to Kafka topics.
- **Apache Kafka**: Acts as a distributed messaging system for handling CDC events.
- **Kafka UI**: Provides a visual interface for observing data flows and troubleshooting.
- **Zookeeper**: Used for coordination and synchronization by Kafka.
- **Apache Spark**: Utilized for real-time processing of data streams from Kafka.

## Docker Compose Setup
The provided Docker Compose file orchestrates the deployment of all necessary services, including MySQL, Debezium, Kafka, Zookeeper, Kafka UI, and Apache Spark.

[Link to Docker Compose file](docker-compose-spark-kafka-mysql.yaml)

## Architecture
The CDC pipeline follows the flow: MySQL ➔ CDC ➔ Debezium ➔ Kafka (Zookeeper) ➔ Apache Spark Streaming
- **MySQL**: Source database where changes occur.
- **Change Data Capture (CDC)**: Utilizes Debezium to monitor database changes.
- **Debezium**: Captures MySQL changes and publishes them to Kafka topics.
- **Kafka**: Acts as the messaging backbone for storing and distributing CDC events.
- **Zookeeper**: Coordinates and synchronizes Kafka brokers.
- **Kafka UI**: Provides a visual tool for monitoring Kafka data flows.
- **Apache Spark Streaming**: Consumes data streams from Kafka for real-time processing and analysis.

## Implementation and Testing
1. Run Docker Compose to deploy the services.
   ```
   docker-compose -f docker-compose-spark-kafka-mysql.yaml up -d
   ```
2. Access MySQL to set up the database.
3. Create a Debezium connector to monitor MySQL and publish to Kafka topics.
4. Configure Debezium and Kafka to monitor changes and publish them.
5. Consume data from Kafka using Spark (PySpark) for real-time processing.

## Kafka UI
Configure Kafka UI for visual monitoring of Kafka data flows.

[Link to Kafka UI config file](kui/config.yml)

Access Kafka UI at: [http://localhost:10000/](http://localhost:10000/)

## Conclusion
Integrating Apache Spark Streaming into the CDC pipeline empowers organizations to extract valuable insights from streaming data in real-time. With Docker Compose facilitating deployment and management, this architecture offers a scalable and efficient solution for real-time data integration and analysis.

**Happy Learning!** ✌️


## Useful commands
```bash
# Start Docker Compose
docker-compose -f docker-compose-spark-kafka-mysql.yaml up -d

# Access MySQL as root user
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec mysql bash -c 'mysql -u root -pdebezium'

# After authentication, execute SQL commands
SELECT user,host FROM mysql.user;

CREATE DATABASE cdc;

GRANT ALL ON cdc.* TO 'debezium'@'%';

FLUSH PRIVILEGES;

SHOW databases;
```

```bash
# Access MySQL as debezium user
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec mysql bash -c 'mysql -u debezium -pdbz'
```

```sql
# Create table in the cdc database
CREATE TABLE cdc.example(
    customerId int,
    customerFName varchar(255),
    customerLName varchar(255),
    customerCity varchar(255)
);

USE cdc;
SHOW tables;
```

### Kafka Connector Debezium
```bash
# Register Debezium connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @./conf/register-mysql.json
```

```bash
# List Kafka topics
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec kafka /kafka/bin/kafka-topics.sh \
--bootstrap-server kafka:9092 \
--list
```

```bash
# Consume data from Kafka topic
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.cdc.example
```

```bash
# Run Spark job
container_id=$(docker ps --filter "name=spark-master" --format "{{.ID}}")
docker exec -it $container_id bash -c 'spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0 /src/real_time_pipeline.py'
```