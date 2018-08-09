# Kafka

Learning's about Kafka

## Why can't two consumers from same consumer group; consume from same partition at the same time in Kafka?

If the consumers belong to the same group, then by design two consumers cannot consume from the same partition. Because this would lead to messages being consumed out of order.

If message1 which is the first message in the chronological order gets picked by consumer1 and message2 which is the second message gets picked by consumer2, it could so happen that consumer1 might take longer time due to failures or retries. Which means message2 would reach first. As this violates kafka’s design principle, thus is not allowed.

Source: http://qr.ae/TUIz49