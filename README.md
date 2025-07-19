# Hadoop ecosystem


## Start the cluster:

Open your terminal in the directory where your `docker-compose.yml` is and run:

```bash
docker compose up -d
```

This will take some time as it downloads all the images and starts the services.

## Verify services:

```bash
docker compose ps
```

Wait until most services show `Up (healthy)` or `Up`. Some health checks might take longer to become healthy.

## Access UIs:

- HDFS NameNode UI: [http://localhost:9870](http://localhost:9870)
- YARN ResourceManager UI: [http://localhost:8088](http://localhost:8088)
- MapReduce JobHistory UI: [http://localhost:8188](http://localhost:8188)
- HiveServer2 Web UI: [http://localhost:10002](http://localhost:10002)
- Spark Master Web UI: [http://localhost:8080](http://localhost:8080)  
  *(Note: This clashes with Trino, you might need to adjust one if you want both web UIs directly accessible from host. For dev, one is usually enough to inspect and the services communicate via their network names).*
- Flink JobManager Web UI: [http://localhost:8081](http://localhost:8081)  
  *(Note: This clashes with Spark Worker, adjust one's exposed port if you need both).*
- Trino Coordinator UI: [http://localhost:8080](http://localhost:8080)  
  *(See note for Spark Master)*

## Interact with services:

### HDFS:

```bash
docker exec -it namenode hdfs dfs -ls /
```

### Kafka:

You can use `kafka-topics.sh`, `kafka-console-producer.sh`, `kafka-console-consumer.sh` by exec'ing into the Kafka container.

```bash
docker exec -it kafka bash
# Inside Kafka container:
kafka-topics.sh --create --topic my_topic --bootstrap-server kafka:9092 --partitions 1 --replication-factor 1
kafka-console-producer.sh --topic my_topic --bootstrap-server kafka:9092
# Type some messages, then Ctrl+D
kafka-console-consumer.sh --topic my_topic --bootstrap-server kafka:9092 --from-beginning
exit
```

### Hive (via Beeline):

```bash
docker exec -it hiveserver2 beeline -u 'jdbc:hive2://hiveserver2:10000/'
# Inside Beeline:
!connect jdbc:hive2://hiveserver2:10000/default;user=hive;password=hive
show databases;
CREATE DATABASE IF NOT EXISTS my_hive_db;
USE my_hive_db;
CREATE TABLE IF NOT EXISTS sample_table (id INT, name STRING) STORED AS ORC;
INSERT INTO sample_table VALUES (1, 'Alice'), (2, 'Bob');
SELECT * FROM sample_table;
!quit
```

### Spark:

You can use `spark-shell`, `pyspark`, or `spark-submit` by exec'ing into `spark-master` or `spark-worker`.

```bash
docker exec -it spark-master bash
# Inside Spark Master container:
spark-shell --master spark://spark-master:7077
# ... or pyspark
pyspark --master spark://spark-master:7077
exit
```

### Flink:

Submit jobs to `flink-jobmanager`. You'd typically build your Flink application (JAR file) and then copy it into the `flink-jobmanager` container or use a mounted volume.

```bash
docker exec -it flink-jobmanager bash
# You'd typically submit a pre-compiled JAR, e.g.:
# flink run -m flink-jobmanager:8081 /path/to/your-flink-job.jar
exit
```

### Trino:

Use the Trino CLI by exec'ing into the `trino-coordinator` container.

```bash
docker exec -it trino-coordinator trino --catalog hive --schema default
# Inside Trino CLI:
show schemas from hive;
show tables from hive.my_hive_db;
SELECT * FROM hive.my_hive_db.sample_table;
```

## Stop and clean up:

```bash
docker compose down -v
```

This command stops all services and removes the containers and associated volumes, ensuring a clean slate for your next session.
