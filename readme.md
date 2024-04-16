```bash
docker-compose -f docker-compose-spark-kafka-mysql.yaml up -d

docker-compose -f docker-compose-spark-kafka-mysql.yaml exec mysql bash -c 'mysql -u root -pdebezium'
```


SELECT user,host FROM mysql.user;

CREATE DATABASE cdc;

GRANT ALL ON cdc.* TO 'debezium'@'%';

FLUSH PRIVILEGES;

SHOW databases;


```bash
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec mysql bash -c 'mysql -u debezium -pdbz'
```
```sql
CREATE TABLE cdc.example(
    customerId int,
    customerFName varchar(255),
    customerLName varchar(255),
    customerCity varchar(255)
);

USE cdc;
SHOW tables;
```

### kafka connector Debezium
```bash
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @./conf/register-mysql.json
```

### 
```bash

docker-compose -f docker-compose-spark-kafka-mysql.yaml exec kafka /kafka/bin/kafka-topics.sh \
--bootstrap-server kafka:9092 \
--list
```

```bash
docker-compose -f docker-compose-spark-kafka-mysql.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic dbserver1.cdc.example
```



```bash
container_id=$(docker ps --filter "name=spark-master" --format "{{.ID}}")
docker exec -it $container_id bash -c 'spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0 /src/real_time_pipeline.py'
```