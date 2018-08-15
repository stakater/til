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

## What is the relation between kafka and kubernetes

A kubernetes pod represents a consumer listening in a consumer group. 

## Where are the offsets stored by consumers

Each consumer stores the offset it read on a special topic in kafka named `__consumer_offsets`. The step of updating offset on this topic is known as commiting the offset. If a consumer fails before commiting the offset of the last read message, it starts from the last saved offset.

## When can the infinite reading loop occur

There are scenarios where a consumer is killed or is reassigned a partition before it can commit the last read offset. In such cases when a new consumer comes to read data from that partition, it will start from the last saved offset which is always the same as new offset is never commited.

## Causes of consumer rebalance

Rebalance is the scenario where partition is assigned to different consumers in the group. 
Rebalancing can happen due to 3 main factors.

Assuming we are talking about Kafka 0.10.1.0 or upwards where each consumer instance employs two threads to function. One is user thread from which poll is called; the other is heartbeat thread that specially takes care of heartbeat things.

session.timeout.ms is for heartbeat thread. If coordinator fails to get any heartbeat from a consumer before this time interval elapsed, it marks consumer as failed and triggers a new round of rebalance.

max.poll.interval.ms is for user thread. If message processing logic is too heavy to cost larger than this time interval, coordinator explicitly have the consumer leave the group and also triggers a new round of rebalance.

heartbeat.interval.ms is used to have other healthy consumers aware of the rebalance much faster. If coordinator triggers a rebalance, other consumers will only know of this by receiving the heartbeat response with REBALANCE_IN_PROGRESS exception encapsulated. Quicker the heartbeat request is sent, faster the consumer knows it needs to rejoin the group.

Suggested values:
session.timeout.ms : a relatively low value, 10 seconds for instance.
max.poll.interval.ms: based on your processing requirements
heartbeat.interval.ms: a relatively low value, better 1/3 of the session.timeout.ms

## Heartbeat and Consumer rebalance

The way consumers maintain membership in a consumer group and ownership of the partitions assigned to them is by sending heartbeats to a Kafka broker designated as the group coordinator (this broker can be different for different consumer groups). 

As long as the consumer is sending heartbeats at regular intervals, it is assumed to be alive, well, and processing messages from its partitions. Heartbeats are sent when the
consumer polls (i.e., retrieves records) and when it commits records it has consumed.

If the consumer stops sending heartbeats for long enough, its session will time out and the group coordinator will consider it dead and trigger a rebalance. If a consumer crashed and stopped processing messages, it will take the group coordinator a few seconds without heartbeats to decide it is dead and trigger the rebalance. During those seconds, no messages will be processed from the partitions owned by the dead consumer. When closing a consumer cleanly, the consumer will notify the group coordinator that it is leaving, and the group coordinator will trigger a rebalance immediately, reducing the gap in processing.

## Does rebalance affects all the topics for a pod?

No, since a pod reads topic after creating a consumer and that consumer becomes a part of a consumer group. In rebalance, the consumer leaves the group and that means the other consumer running in the pod that is reading to other message is not affected by this reshuffle and it will keep on working as expected.

## How to fix infinite consumer rebalancing loop problem

One of the cause of rebalancing is because a consumer cannot finish reading the messages within the max interval. We can fix this issue by increasing the timeout interval property for a consumer and decreasing the total number of records in a poll.
This change will allow consumers to read to messages within alloted time and commit the offset without reshuffle.

Here are the properties to change for a consumer `max.poll.records` and `max.poll.interval.ms`. Default values for these properties are `500` and `300000` respectively.

## Tips to diagnose the kafka looping issue through logging

- Make sure to log offset and partition on the consumer at listener event.

- Enable debug level logging in your consumer application that you want to debug.

- For the topic partition that is giving errors, go to kafka manager and check if there is lag which shows the messages are not being consumed by the consumer

- Search for the topic in kibana and notice if you see any messages without error. This check will help you determine if all the messages on that partition of that topic is throwing error. To do this you need to search for this query in kibana 
`kubernetes.namespace_name: "dev" && "offset:" && "lead-v4" && "partition = 1" && "nonScaniaChassisMaintenanceOccasion" -"log:error"`.
Here `lead-v4` is the app name and `nonScaniaChassisMaintenanceOccasion` is the topic name.

- Search for this query `+"OffsetAndMetadata" +"nonScaniaChassisMaintenanceOccasion-1"` for a pod and an app and you will notice that the commited offset for the topic and partition gets reset after after incrementing 500 times. Which shows that the consumer is stuck in a loop and the offset is resetting. 

5- Also in the logs you will notice that you will also find `org.apache.kafka.clients.consumer.CommitFailedException` exception which means the commit of offset was not possible. This exception comes right before rebalance and stops the broker from commited the last read message offset.

6- Another keyword that points to this issue is to search for `o.a.k.c.c.internals.ConsumerCoordinator` and `Revoking previously assigned partitions` keywords for your pod which shows partitions are revoked.

7- Under `o.a.k.c.c.internals.AbstractCoordinator` look for `(Re-)joining group` which shows the consumer is rejoining the group after reshuffle.


