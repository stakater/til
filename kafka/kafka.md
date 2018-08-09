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
