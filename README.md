# manually run zookeeper and kafka from terminal

curl "https://downloads.apache.org/kafka/2.6.3/kafka_2.13-2.6.3.tgz" -o ~/Downloads/kafka.tgz
mkdir ~/kafka && cd ~/kafka
tar -xvzf ~/Downloads/kafka.tgz --strip 1

/bin/sh bin/zookeeper-server-start.sh config/zookeeper.properties
/bin/sh bin/kafka-server-start.sh config/server.properties

# run both services from a single docker compose file

docker-compose up

### run zookeeper from image

docker run -d \
--name zookeeper \
--hostname zookeeper \
-p 2181:2181 \
--env ZOOKEEPER_CLIENT_PORT=2181 \
-e ZOOKEEPER_TICK_TIME=2000 \
confluentinc/cp-zookeeper:5.3.1

echo stat | nc 127.0.0.1 2181
echo mntr | nc 127.0.0.1 2181
echo isro | nc <zookeeper_ip> 2181

### run kafka from image

Zookeeper_Server_IP=$(docker inspect zookeeper --format='{{ .NetworkSettings.IPAddress }}')

docker run -d \
--interactive \
--name kafka \
--hostname kafka \
-p 9092:9092 \
--env KAFKA_ZOOKEEPER_CONNECT=${Zookeeper_Server_IP}:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092 \
-e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT \
-e KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT \
-e KAFKA_BROKER_ID=1 \
-e KAFKA_DEFAULT_REPLICATION_FACTOR=1 \
-e KAFKA_NUM_PARTITIONS=3 \
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
confluentinc/cp-kafka:5.3.1

<!-- docker run \
--rm confluentinc/cp-kafka:5.3.1 kafka-topics.sh \
--create \
--topic first-topic \
--replication-factor 1 \
--partitions 1 \
--zookeeper 127.0.0.1:2181 -->

<!-- docker run \
--rm confluentinc/cp-kafka:5.3.1 kafka-topics.sh \
--list \
--zookeeper 127.0.0.1:2181 -->

<!-- docker run --rm --interactive \
confluentinc/cp-kafka:5.3.1 kafka-console-producer.sh \
--topic first-topic \
--broker-list 127.0.0.1:9092 -->

<!-- docker run --rm \
confluentinc/cp-kafka:5.3.1 kafka-console-consumer.sh \
--topic first-topic \
--from-beginning \
--zookeeper 127.0.0.1:2181 -->

docker exec -it zookeeper bash
docker exec -it kafka bash

sudo apt-get install kafkacat
