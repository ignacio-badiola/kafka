# Description

Simple Kafka 2.5.0 project 

## Setup

Follow instructions from [kafka-setup](https://github.com/ignacio-badiola/kafka/wiki/Kafka-setup)


## Optional kafka tools:
- [Conduktor](https://www.conduktor.io/): a Kafka GUI, to make the development and management of Apache Kafka clusters as easy as possible
- [kafkacat](https://github.com/edenhill/kafkacat): open-source alternative to using the Kafka CLI
  - See post: [Debugging with kafkacat](https://medium.com/@coderunner/debugging-with-kafkacat-df7851d21968)

## Work with TOPIC
- CREATE A TOPIC: `kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1`
- OTHER TOPIC OPTIONS: kafka-topics.sh --bootstrap-server localhost:9092 --describe //  --list // --delete

## Work with PRODUCER

### Producer: send message to a topic

Send messages to the stream using CMD: 
  - `kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic`  

### Producer: send message to a topic using KEYS

Send message to first_topic using keys separated by `,`
  - `kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic --property parse.key=true --property key.separator=,`

`NOTE`: If we send message to a topic that doesnt exists then the topic will be automatically created with default partitions and replication-factor values

## Work with CONSUMERS

### Work with PARTITIONS and OFFSETS

#### CASE 1: Consume from DIFFERENT TOPIC in DIFFERENT PARTITION => each consumer will get the message sent to the topic/partition
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0` (same as --from-beginning)
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --partition 1 --from-beginning`

#### CASE 2: Consume from SAME TOPIC in DIFFERENT PARTITION => each consumer will get the message sent to the topic in order from the selected partition
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0` (same as --from-beginning)
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 1 --from-beginning`

#### CASE 3: Consume from SAME TOPIC in SAME PARTITION => each consumer will get the SAME message sent to the topic from the selected partition
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0` (same as --from-beginning)
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --from-beginning`

### Work with CONSUMERS GROUPS

`IMPORTANT: You cannot work with groups and partitions at the same time`

#### CASE 1: DIFFERENT GROUPS consume from DIFFERENT TOPIC => each group will get the message sent to the topic
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --group my-second-app-consumer --from-beginning`
  
#### CASE 2: DIFFERENT GROUPS consume from SAME TOPIC => each group will get same messsages in both groups
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-second-app-consumer --from-beginning`

#### CASE 3: SAME GROUPS consume from SAME TOPIC => messages will be load balanced into the consumers
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`
- `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`

#### CASE 4: SAME GROUPS consume from SAME TOPIC => Each consumer will read messages from mutually-exclusive partitions
- Consumer_1: `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`
- Consumer_2: `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning`

If we have 3 partitions, then one possible partition assignment is:
  - Consumer_1: first_topic-0, first_topic-2
  - Consumer_2: first_topic-1

If we create another consumer for the same topic and same group, then consumers group will be `rebalanced`.
Here's a possible partition assignment after rebalancing the group:
  - Consumer_1: first_topic-2
  - Consumer_2: first_topic-1
  - Consumer_3: first_topic-0


### Consumer: receive message from a topic using KEYS

Receive message to first_topic using keys separated by `,`
  - `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --from-beginning --property print.key=true --property key.separator=,`


`IMPORTANT NOTE:`
  - Running consumers by partition (--partition X) will receive: 
    - if message have key => ordered message in from the same defined partition
    - if message does NOT have key => message sent to the defined partition by using round robing LB from producer
  - Running consumers by group (--group my-X-grooup), each group will receive the same message

#### Describe all defined groups

Lets say that we have two groups consuming from same topic (first_topic):
  - my-first-app-consumer (paused)
  - my-second-app-consumer (running)


We needto run the following cmxd: 
```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --all-groups
```

As result we will get:
```
GROUP                 TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                                           HOST            CLIENT-ID
my-first-app-consumer first_topic     0          35              35              0               consumer-my-first-app-consumer-1-4f37ab3f-2e57-4317-a38d-fba55a836371 /192.168.1.3    consumer-my-first-app-consumer-1
my-first-app-consumer first_topic     1          40              40              0               consumer-my-first-app-consumer-1-4f37ab3f-2e57-4317-a38d-fba55a836371 /192.168.1.3    consumer-my-first-app-consumer-1
my-first-app-consumer first_topic     2          44              44              0               consumer-my-first-app-consumer-1-4f37ab3f-2e57-4317-a38d-fba55a836371 /192.168.1.3    consumer-my-first-app-consumer-1

Consumer group 'my-second-app-consumer' has no active members.

GROUP                  TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my-second-app-consumer first_topic     0          35              35              0               -               -               -
my-second-app-consumer first_topic     1          40              40              0               -               -               -
my-second-app-consumer first_topic     2          44              44              0               -               -               -
```

### Work with OFFSETS

#### To RESET offsets we can run the following cmd:
- `kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-app --topic first_topic --reset-offsets --to-earliest --execute`

Expected result: 
```
GROUP                          TOPIC                          PARTITION  NEW-OFFSET     
my-first-app                   first_topic                    0          0              
my-first-app                   first_topic                    1          0              
my-first-app                   first_topic                    2          0   
```

#### To RESET offsets we can run the following cmd:
- `kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-app --topic first_topic --reset-offsets --shift-by 4 --execute`

Expected result: 
```
GROUP                          TOPIC                          PARTITION  NEW-OFFSET     
my-first-app                   first_topic                    0          4              
my-first-app                   first_topic                    1          4              
my-first-app                   first_topic                    2          4  
```

## What is next...?

Reference: [Producer vs Consumer vs Kafka Connect vs Kafka Streams vs KSQL](https://medium.com/@stephane.maarek/the-kafka-api-battle-producer-vs-consumer-vs-kafka-connect-vs-kafka-streams-vs-ksql-ef584274c1e)

### Kafka Streams

- `Definition:` Data processing and transformation JAVA library within Kafka
- `Use cases:` Data transformations from **one topic to another**, data enrichment, fraud detection, monitoring and alerting
- `Caracteristics`:
  - Standard JAVA Application (no need another cluster to run)
  - Highly scalable, elastic and fault tolerant
  - One record at a time processing (no batch processing SPARK will do this)
  - 



### Kafka Registry

- Confluent schema registry
- Communicate with producers and consumers
- Apache AVRO as data format instead of JSON (some learning curve)
- `PROS:` Decrease size of payload of data sent to Kafka


## GUIDELINES: Partitions, Replication factors and Cluster definition

### Partitions guidelines

- Each partition can handle a throughput of few MB/s
- Number of partitions is the max numbers of **consumers** that we can have
- More partitions implies:
  - **PROS:**
    - Better parallelism, therefor better throughput
    - Leads to be able to use **more consumers**
    - Create as many **brokers** as partitions to avoid idle brokers
  **- CONS:**
    - more elections to perform from ZOOKEEPER
    - more files opened by KAFKA

Guidelines:
- How many **partitions** per **topic**?
  - Small cluster (< 6 brokers) then number of Partitions = 2 x Brokers
  - Big cluster (> 12 brokers) then number of Partitions = Brokers
- How many consumers will i need to run in parallel on throughput peak time?
  - If the number is 20 (as an example) then we should have 20 partitions, so each consumer have one partition to pull data from

### Replication factors guidelines

- Should be at least 2, recommended 3 and maximum 4
- If we have `N` replication factors then:
  - **PROS:** `N - 1` brokers can fail
  - **CONS:** 
    - Higher replication factor leads to more latency when `acks=all`
    - Higher replication factor more disk usage needed

Guidelines:
- Always set it to 3 to get started (we need one broker for each replication factor)
- If performance is an issue then improve brokers (servers) but **DO NOT** sacrifice replication factor
- NEVER EVER set replication factor to 1

### Clusters guidelines

- A `Broker` should **NOT** hold more than 2K to 4K partitions (across all topics)
- A `Cluster` **NOT** hold more than 20K partitions across all brokers. When leader goes down ZOOKEEPER must perform leader selection degrades performance
- If we need more partitions then add more brokers
- If we need more than 20K partitions then use NETFLIX model by adding more clusters

### Producers guidelines

ACKS=all must be used with `min.insync.replicas`.
If we set `min.insync.replicas = 2` means that at least 2 brokers (leader + 1 ISR) must respond that they have the data sent.
In this scenario if all ISR are down and only the leader receives the message then it will respond with **NotEnoughReplicasException** that needs to be **handled by producer**.

In such case we can use `retries` setting:
  - default to 0 on kafka <= 2.0
  - default to max_int on kafka > 2.0
And there is also `retry.backoff.ms` setting set to 100ms by default

#### Idempotent Producer
The producer can introduce duplicate messages due to retries when an ack from kafka did not reach the producer on time. Producer will send the same message twice.
To solve this producer send an id set to each message and this way kafka knows that already have this message and it will only respond ack but NOT commit the message to the stream.

To ensure order messaging, besides of `key` to deliver the message with same key to same partition, we need to set `max.in.flight.requests.per.connection = 1`. This might impact throughput but ensure ordering

There's also a timeout called `delivery.timeout.ms` (defaults to 2 minutes) time the producer will expect a response after each request.

#### Summary config to create a safe producer
- ENABLE_IDEMPOTENCE_CONFIG: true
- ACKS_CONFIG: all
- RETRIES_CONFIG: MAX_INTEGER
- MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION = 5 (set to 1 if ordering needs to be ensured)
- LINGER.MS = 20ms (number of miliseconds to wait for more messages to be sent in a batch, saves requests. Default: 5ms)
- BATCH.SIZE = 32KB (max number of bytes to be sent to kafka Default: 16KB)
- COMPRESION_TYPE_CONFIG = [snappy](https://github.com/google/snappy)

### Naming Convention

[Reference to naming convention](https://riccomini.name/how-paint-bike-shed-kafka-topic-naming-conventions)

## Case Studies

**Scenario:**
  - 3 brokers
  - 1 topic
  - 2 partitions
  - 2 replication factor

**Task 1:** Increase partition count after processing lifecycle already started
**Effect 1:** Break key order guarantees among partitions

**Task 2:** Increase replication factor after processing lifecycle already started
**Effect 2:** Add more pressure to cluster to support this channge and will degrade performance or inestability
