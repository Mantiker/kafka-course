# Kafka

Real-time (latency <10 ms)  data streaming platform

1) Based on Kafka infra
2) Implemented Kafka Logic part

## Kafka Cluster

Isolated kafka platform on 1 or few severs. Can use cloud, on-premise, etc.

1) Contains Topics
2) Comopsed of multiple brokers (servers)

### Architecture

#### Kafka Broker

1) Broker is server
2) Identified by ID (integer)
3) Contains certain kafka topic partitions
4) After connacting to any broker (calld a bootstrap broker), you will be connected to the entire cluster (kafka clients have smart mechanics for that)
5) good to start - 3 brokers, some clusters could have over 100 brokers

#### Producers

Producers write data to topics (which are made of partitions)

1) Producers know to which partition to write to (to partition leader only, and which Kafka broker has it)
2) In case of Kafka broker failures, Producer will automatically recover
3) Producer can send a key with the message (with different key complexity - string, number, binary, etc)
4) If key is null, data is sent round robin (partition 0, then 1, then 2, etc)
5) If key is NOT null, data for the same key sent always to the same partition (based on hashing strategy)
6) Message ordering in partition based on key
7) Has default serializers: srting/json, int/float, avro, protobuff, ...
8) Producers can choose to receive acknowledgement of data writes:
    - acks = 0: producer won't wait for acknowledgement (possible data loss)
    - acks = 1: producer will wait for leader acknowledgement (limited data loss)
    - acks = all: Leader + replicas acknowledgement (no data loss)

#### Consumers

Consumers read data from a topic - pull model

1) Consumers automatically know which broker to read from (from leader by default)
2) In case of broker failure consumers know how to recover
3) Data is read in order frmo low to high offset within each partition
4) Consumer uses desirializers on key and value of message
5) Has common desirializers: srting/json, int/float, avro, protobuff, etc

#### Consumer Groups

1) All the consumers in an application read data as a consumer groups
2) Each consumer within a group reads from exclusive partitions
3) If you have more consumers than partitions, some consumers will be inactive
4) It acceptable to have multiple comnsumer groups on the same topic
5) Consumer group identifies by using consumer property _group.id_
6) Kafka stores the offsets at which each consumer group has been reading in topic ___consumer_offsets_
7) When consumer has processed data received from Kafka, it should periodically commiting the offsets (the kafka broker will write to __consumer_offset, not the group itself)

#### Delivery semantics for consumer

1) At least once (usually prefered)
    - Offsets are commited after the message will be read again
    - If the processing goes wrong, the message will be read again
    - This can result in duplicate processing of messages. Make sure your processing is idempotent (i.e. processing again the messages won't impact your system)
2) At most once
    - Offsets are commited as soon as messages are received
    - If the processing goes wrong, some messages will be lost (they won't be read again)
3) Exactly once
    - For Kafka => Kafka workflows: use the Transactional API (easy with Kafka Streams API)
    - For Kafka => External System worflows: use an idempotent consumer

By default, consumer will automatically commit offsets (at leas once)



### Logic part

#### Kafka Topic

A particular stream of data

1) Identified by name
2) Supports any kind of message format (json, binary, etc)
3) Split in partitions
4) One topic (his partitions) are destributed among brokers based on partitions param
5) Contains copy of partitions are based on replication-factor param
6) Data is kept only for a limited time (default - one week, configurable)
7) Kafka topic data is immutable (can't be changed - writed in "kafka log")

#### Partition

1) Represents topic partialy
2) For given partition only one broker could be a leader
3) Has a leader and multiple replicas (ISR -> in-sync replica)
3) Messages in each partition ordered
4) Each mesage in partition has id (offset) => id is not a Key
5) Each partiton can have different current offset (offset only have a meaning for a specific partition)
6) Data is assigned randomly to a partition unless a key is provided
7) If data has a key (producr posts key + message) different messages with the same key always go through same partition. Order is guaranteed only within a partition (not across partitions)

#### Messages

The data which is sent by producer, gone through partition, took by consumer

Contains:

1) Key - binary (can be null) - serialized/unserialized on producer/consumer side
2) Value - binary (can be null) - serialized/unserialized on producer/consumer side
3) Compression type (none, gzip, snappy, lz4, zstd)
4) Headers (optional): Key: Value
5) Partition + Offset
6) Timestamp (system or user set)

Data types of key and value must not be changed furing a topic lifecycle (create a new topic instead)

