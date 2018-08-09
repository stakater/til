# Kafka

Learning's about Kafka

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

