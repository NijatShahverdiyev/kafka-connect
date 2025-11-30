# Kafka Connect — README

1. run docker-compose up -d
2. docker ps -- find mongo <container_id>
3. docker exec -t <container_id> mongosh --eval "rs.initiate()"
    - This initiates mongo replicates, as kafka connect only works with mongodb replicaset
4. connect mongodb using compass or something with below connection url:
    - <mongodb://localhost:27017/product-db?directConnection=true&replicaSet=rs0>
5. curl -X POST -H "Content-Type: application/json" \
        --data @connector.json \
        http://localhost:8083/connectors
6. add document to mongodb then kafka-connector will send it to kafka topic


- connector.json
  - Defines a MongoDB source connector named `mongodb-connectors` using the `com.mongodb.kafka.connect.MongoSourceConnector` class.
  - Connects to the Mongo replica set at `mongo:27017/product-db`, watching the `product-db.products` collection.
  - Filters change stream events to `CARD` documents with `data.status` of `Ready`, projecting only `userId` and `cardNumber` fields; publishes full documents only and looks up full doc on update.
  - Writes matching events to Kafka topic `products.db.card-event-test` with string key/value converters; routes errors to DLQ topic `products.db.kafka-connector-dlt-test`.
  - Enables error logging/tolerance, small poll batches (1 message, 20ms wait), and basic schema/key formatting for the connector output.

- docker-compose.yaml
  - Spins up Zookeeper and a single Kafka broker (PLAINTEXT listeners on `9092` inside and `29092` for host).
  - Runs Kafka Connect with plugin path mounting `./plugins` into `/etc/kafka-connect/jars`; sets JSON converters without schemas and standard internal topics.
  - Starts MongoDB 4.4.9 in replica set mode (`rs0`) with persisted volume `db_data`.
  - Adds Kafka-UI (kafbat) on host port `8090` preconfigured to talk to Kafka (`kafka:9092`) and the Connect REST API (`connect:8083`).
  - All services share the `shared-net` bridge network so container hostnames resolve for intra-cluster communication.
