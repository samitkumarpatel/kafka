# schema-registry


Start Kafka

```sh
docker run --rm \                                                                                                                                                                                          130 ↵
  --name=kafka-confluent \
  --network=kafka \
  -p 9092:9092 \
  -e CLUSTER_ID=4L6g3nShT-eMCtK--X86sw \
  -e KAFKA_LISTENERS="PLAINTEXT://kafka-confluent:29092,PLAINTEXT_DOCKER_HOST://kafka-confluent:39092,PLAINTEXT_HOST://0.0.0.0:9092,CONTROLLER://kafka-confluent:9094" \
  -e KAFKA_ADVERTISED_LISTENERS="PLAINTEXT://kafka-confluent:29092,PLAINTEXT_HOST://localhost:9092,PLAINTEXT_DOCKER_HOST://kafka-confluent:39092" \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP="PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT,CONTROLLER:PLAINTEXT,PLAINTEXT_DOCKER_HOST:PLAINTEXT" \
  -e KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT \
  -e KAFKA_PROCESS_ROLES="broker,controller" \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_NODE_ID=1 \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS="1@kafka-confluent:9094" \
  -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
  -e KAFKA_OFFSETS_TOPIC_NUM_PARTITIONS=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1 \
  -e KAFKA_TRANSACTION_STATE_LOG_MIN_ISR=1 \
  -e KAFKA_LOG_FLUSH_INTERVAL_MESSAGES=9223372036854775807 \
  -e KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0 \
  confluentinc/cp-kafka:latest
```

Start Schema Registry

```sh
docker run --rm --name kafka-schema-registry --network kafka -p 8081:8081 -e SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS=PLAINTEXT://kafka-confluent:39092 -e SCHEMA_REGISTRY_HOST_NAME=schema-registry confluentinc/cp-schema-registry:latest
```