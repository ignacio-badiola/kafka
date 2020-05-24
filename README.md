# Description

Simple Kafka 2.5.0 project 

## Install and configs
- Download stable version of kafka
- mv ~/Downloads/kafka_2.12-2.5.0.tgz .
- tar xvf kafka_2.12-2.5.0.tgz  
- cd kafka_2.12-2.5.0
- mkdir data
- mkdir data/zookeeper
- mkdir data/kafka
- nano config/server.properties 
     => under Logs Basics to: log.dirs=/Users/mac-64/Documents/Learning/Kafka/kafka_2.12-2.5.0/data/kafka
     => under Logs Basics to: num.partitions=3 // Default partition values to be created when we produce to a topic that doesn't exist
- nano config/zookeeper.properties  => change the directory where the snapshot is stored: dataDir=/Users/mac-64/Documents/Learning/Kafka/kafka_2.12-2.5.0/data/zookeeper

## Start servers
- START ZOOKEEPER: zookeeper-server-start.sh config/zookeeper.properties  
- START KAFKA: kafka-server-start.sh config/server.properties 

## Work with TOPIC
- CREATE A TOPIC: kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1
- OTHER TOPIC OPTIONS: kafka-topics.sh --bootstrap-server localhost:9092 --describe //  --list // --delete

## Work with PRODUCER
- PRODUCER - Send cmd messages to the stream: kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic  
           - NOTE: If we send message to a topic that doesnt exists then the topic will be automatically created with default partitions and replication-factor values

## Work with CONSUMER

### Work with PARTITIONS and OFFSETS

#### CASE 1: Consume from DIFFERENT TOPIC in DIFFERENT PARTITION => each consumer will get the message sent to the topic/partition
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0 (same as --from-beginning)
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --partition 1 --from-beginning

#### CASE 2: Consume from SAME TOPIC in DIFFERENT PARTITION => each consumer will get the message sent to the topic in order from the selected partition
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0 (same as --from-beginning)
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 1 --from-beginning

#### CASE 3: Consume from SAME TOPIC in SAME PARTITION => each consumer will get the SAME message sent to the topic from the selected partition
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --offset 0 (same as --from-beginning)
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --partition 0 --from-beginning


### Work with GROUPS

`IMPORTANT: You cannot work with groups and partitions at the same time`

#### CASE 1: DIFFERENT GROUPS consume from DIFFERENT TOPIC => each group will get the message sent to the topic
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_topic --group my-second-app-consumer --from-beginning
  
#### CASE 2: DIFFERENT GROUPS consume from SAME TOPIC => each group will get same messsages in both groups
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-second-app-consumer --from-beginning

#### CASE 3: SAME GROUPS consume from SAME TOPIC => messages will be load balanced into the consumers
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning

#### CASE 4: SAME GROUPS consume from SAME TOPIC => messages will be load balanced into the consumers
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning
- kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --group my-first-app-consumer --from-beginning

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
