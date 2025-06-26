# Kafka all-in-one

### confluentinc
- [confluent github with README's](https://github.com/confluentinc)
- [confluent all-in-one example like run with docker, docker compose and ect...](https://github.com/confluentinc/cp-all-in-one/tree/control-center)
- [confluent example](https://github.com/confluentinc/examples)

---

### apache
- [Docs](https://kafka.apache.org/).
- [Quick Start](https://kafka.apache.org/quickstart). 

<details>

<summary>Initial Example with apache/kafka-native.</summary>

> To know more about this image [explore here](https://hub.docker.com/r/apache/kafka-native).


- Start Kafka

```sh
# start apache/kafka-native
docker run --name kafka-native -p 9092:9092 apache/kafka-native:4.0.0
```
- Download cli for administrate
> Download Kafka cli - as apache/kafka-native does not contain those 

[Download](https://kafka.apache.org/documentation/quickstart_download)

- PreReq
```sh
# Create a topic
kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
# Describe an existing topic
```

- Consumer
```sh
# Consumer-1
kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092 --group Tax.report

# Consumer-2
kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092 --group Tax.audit

```

- Producer
```sh
kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
# It will open a prompt then type your message there.
```

- With Kafka Connect
Import/export your data as streams of events with Kafka Connect

[Under Construction...](https://kafka.apache.org/quickstart#quickstart_kafkaconnect)

</details>

<details>
<summary>Example of schema-registry with  apache/kafka-native OR confluentinc/cp-kafka & confluentinc/cp-schema-registry.</summary>

> To learn more about `confluentinc/cp-schema-registry` , follow [this guide](https://github.com/confluentinc/schema-registry)

To start and use follow above apache/kafka-native Guide.

... It's for confluentinc/cp-schema-registry only.

[Installation Guide](https://docs.confluent.io/platform/current/schema-registry/installation/index.html)

- Start Kafka server and schema registry
```sh
# Create a network
docker network create kafka

# run kafka-native
docker run --rm --name kafka-native -p 9092:9092 --network kafka apache/kafka-native:4.0.0

# OR
# run confluentic/cp-kafka
docker run --rm \
  --name=kafka-confluent \
  --network=kafka \
  -p 9092:9092 \
  -e CLUSTER_ID=4L6g3nShT-eMCtK--X86sw \
  -e KAFKA_LISTENERS="PLAINTEXT://kafka-confluent:29092,PLAINTEXT_HOST://0.0.0.0:9092,CONTROLLER://kafka-confluent:9094" \
  -e KAFKA_ADVERTISED_LISTENERS="PLAINTEXT://kafka-confluent:29092,PLAINTEXT_HOST://localhost:9092" \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP="PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT,CONTROLLER:PLAINTEXT" \
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

# Start schema registry

docker run --rm --name kafka-schema-registry --network kafka -p 8081:8081 -e SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS=PLAINTEXT://kafka-confluent:29092 -e SCHEMA_REGISTRY_HOST_NAME=schema-registry confluentinc/cp-schema-registry:latest

```
- Create Topic

```sh
kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

> [Find usage of schema-registry](https://github.com/confluentinc/schema-registry?tab=readme-ov-file#quickstart-api-usage-examples)

- Version some schema in schema registry.

```sh
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\": \"int\"}"}' http://localhost:8081/subjects/quickstart-events-key/versions

curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\": \"boolean\"}"}' http://localhost:8081/subjects/quickstart-events-value/versions

# Check the version
curl -X GET http://localhost:8081/subjects/quickstart-events-key/versions
curl -X GET http://localhost:8081/subjects/quickstart-events-value/versions

# Check the schema based on version
curl -X GET http://localhost:8081/subjects/quickstart-events-key/versions/1
curl -X GET http://localhost:8081/subjects/quickstart-events-value/versions/1
```

- Producer

> Note that `kafka-avro-console-producer` not part of kafka binary. It's part of schema-registry binary. Make sure you have it.

```sh
kafka-avro-console-producer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property value.schema='{"type":"boolean"}'
> true
> false
> abc - Will fail to produce

# Produce key:value
kafka-avro-console-producer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property value.schema='{"type":"boolean"}' --property key.schema='{"type":"int"}' --property parse.key=true --property key.separator=:
> 1:true
> 2:false
> 3:abc - will fail
```

- Consumer

```sh
# consumer-1
kafka-avro-console-consumer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property print.key=true --group a

# Consumer-2
kafka-avro-console-consumer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property print.key=true --group b
```

- Test with wrong input

```sh
kafka-avro-console-producer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property value.schema='{"type":"string"}' --property key.schema='{"type":"float"}' --property parse.key=true --property key.separator=:

# This will error out with >>>>>>>> Caused by: io.confluent.kafka.schemaregistry.client.rest.exceptions.RestClientException: Schema being registered is incompatible with an earlier schema for subject "quickstart-events-key", details: [{errorType:'TYPE_MISMATCH', description:'The type (path '/') of a field in the new schema does not match with the old schema', additionalInfo:'reader type: BOOLEAN not compatible with writer type: INT'}, {oldSchemaVersion: 1}, {oldSchema: '"int"'}, {validateFields: 'false', compatibility: 'BACKWARD'}]; error code: 409
```
- Make the wrong test work.

```sh
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\": \"float\"}"}' http://localhost:8081/subjects/quickstart-events-key/versions

# Producer
kafka-avro-console-producer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property value.schema='{"type":"boolean"}' --property key.schema='{"type":"int"}' --property parse.key=true --property key.separator=:
> 1:true
> 3:false

kafka-avro-console-producer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property value.schema='{"type":"boolean"}' --property key.schema='{"type":"float"}' --property parse.key=true --property key.separator=:
> 1.3:false
> 3.7:false
> 1:true
> 0.6:false

# Consumer
kafka-avro-console-consumer --topic quickstart-events --bootstrap-server PLAINTEXT://kafka-confluent:29092 --property schema.registry.url=http://localhost:8081 --property print.key=true --group a
1	true
3	false
1.3	false
3.7	false
1.0	true
0.6	false
```



</details>
