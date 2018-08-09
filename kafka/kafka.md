# Kafka

Learning's about Kafka

## To log offset and partition numbers on Java Consumer

    @StreamListener(Sink.INPUT)
    public void handle(
            @Payload final [TOPIC_CLASS] [TOPIC_CLASS_OBJECT],
            @Header(KafkaHeaders.OFFSET) final Long offset,
            @Header(KafkaHeaders.RECEIVED_PARTITION_ID) final Integer partition) {

        LOGGER.info("Receiving message (partition:{}, offset:{})", partition, offset);
    }
    
Here [TOPIC_CLASS] and [TOPIC_CLASS_OBJECT] are to be replaced by actual class and object respectively 

## log levels to set to TRACE

For debugging issues in kafka consumers, set following log levels for various properties

    logging.level.org.apache.kafka.clients.consumer.internals.AbstractCoordinator: TRACE
    logging.level.org.springframework.kafka: DEBUG
    logging.level.org.apache.kafka: INFO

## Useful  logs to look for when debugging kafka consumer issues
     
     o.a.k.clients.consumer.ConsumerConfig
This log shows the current configurations of the consumer.
    
    o.s.c.s.b.k.KafkaMessageChannelBinder
This log is used to check when a partition was assigned to a consumer and when it was revoked

    o.s.kafka.listener.LoggingErrorHandler
LogggingErrorHandler logs errors when an error occurs while reading a message on consumer side

    o.a.k.c.c.internals.AbstractCoordinator
Logs actions by the coordinator

## Useful settings for kafka consumer

    session.timeout.ms
The amount of time a consumer can be out of contact with the brokers while still considered alive defaults to 3 seconds.
    
    enable.auto.commit
This parameter controls whether the consumer will commit offsets automatically, and defaults to true. 

    max.poll.records
This controls the maximum number of records that a single call to poll() will return. Defaults to 500

    max.poll.interval.ms
The maximum delay between invocations of poll() when using consumer group management. This places an upper bound on the amount of time that the consumer can be idle before fetching more records. If poll() is not called before expiration of this timeout, then the consumer is considered failed and the group will rebalance in order to reassign the partitions to another member. Defaults to 300000 (5 minutes)

## kafka rest
Kafka is a rest proxy to kafka provided by confluent. It enables a rest interface to interact with the kafka broker and zookeeper. 
There are saveral usecases for kafka rest some of them include 

- posting a message to a topic on a certain partition.
- creating a consumer group to read data from a topic
- getting the total number of topics

For more information: https://docs.confluent.io/current/kafka-rest/docs/api.html

## Why can't two consumers from same consumer group; consume from same partition at the same time in Kafka?

If the consumers belong to the same group, then by design two consumers cannot consume from the same partition. Because this would lead to messages being consumed out of order.

If message1 which is the first message in the chronological order gets picked by consumer1 and message2 which is the second message gets picked by consumer2, it could so happen that consumer1 might take longer time due to failures or retries. Which means message2 would reach first. As this violates kafka’s design principle, thus is not allowed.

Source: http://qr.ae/TUIz49

## How does Kafka scale consumers?

Kafka scales consumers by partition such that each consumer gets its share of partitions. A consumer can have more than one partition, but a partition can only be used by one consumer in a consumer group at a time. If you only have one partition, then you can only have one consumer.

## What are leaders? Followers?

Leaders perform all reads and writes to a particular topic partition. Followers replicate leaders.

## How does Kafka perform failover for consumers?

If a consumer in a consumer group dies, the partitions assigned to that consumer is divided up amongst the remaining consumers in that group.

## How does Kafka perform failover for Brokers?

If a broker dies, then Kafka divides up leadership of its topic partitions to the remaining brokers in the cluster.

## What is an ISR?

An ISR is an in-sync replica. If a leader fails, an ISR is picked to be a new leader.

## What is a consumer group?

A consumer group is a group of related consumers that perform a task, like putting data into Hadoop or sending messages to a service. Consumer groups each have unique offsets per partition. Different consumer groups can read from different locations in a partition.

## Does each consumer group have its own offset?

Yes. The consumer groups have their own offset for every partition in the topic which is unique to what other consumer groups have.

## When can a consumer see a record?

A consumer can see a record after the record gets fully replicated to all followers.

## What happens if there are more consumers than partitions?

The extra consumers remain idle until another consumer dies.

## What happens if you run multiple consumers in many threads in the same JVM?

Each thread manages a share of partitions for that consumer group.

## What is the different message delivery semantics?
 
There are three message delivery semantics: at most once, at least once and exactly once.

## How would you prevent a denial of service attack from a poorly written consumer?

Use Quotas to limit the consumer’s bandwidth.

## What is the default producer durability (acks) level?

All. Which means all ISRs have to write the message to their log partition.

## What happens by default if all of the Kafka nodes go down at once?

Kafka chooses the first replica (not necessarily in ISR set) that comes alive as the leader as unclean.leader.election.enable=true is default to support availability.

