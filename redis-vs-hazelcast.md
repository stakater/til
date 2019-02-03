# Redis vs Hazelcast

This document compares Redis with Hazelcast.

## Hazelcast

### 1. Benchmark

Read these two reports ofc from hazelcast so, might be biased:

- https://hazelcast.com/resources/benchmark-redis-vs-hazelcast/
- https://hazelcast.com/resources/whitepaper-redis-comparison/

They clearly show Hazelcast beats Redis!

### 2. Community

Quite vibrant and awesome

https://hazelcast.org/get-involved/

Just sign up on gitter and then you can have chat with real users & the core developers!

### 3. Operations

Fairly simple; have used in really large scale projects and never had even single incident with Hazelcast.

Don't have any operator in place; but we are planning to write one oursevles. Since they have [official auto-discovery plugin](https://github.com/hazelcast/hazelcast-kubernetes) 
it is very easy to setup cluster on kubernetes or OpenShift 

### 4. Development

Given (Java) Spring it just abstracts away the under lying technology; so, basically no difference for developers.

### 5. Wins

- Community is awesome and you get help on gitter right away!
- If you are Java shop then Hazelcast is unique in that it can be embedded in a Java host process, making it great for building stateful microservices without an external database dependency. It also has some other small differences, like the ability to get a call back from a key expiration. In a sense, it does less overall but the few things it does, it does them better. Especially if you're using Java.

## Redis

### 1. Benchmark

Some benchmark reports from Redis; so could be biased:

- https://redislabs.com/blog/benchmarking-redis-enterprise-5-2-0-vs-hazelcast-3-9/

Also **note** this is Redis Enterprise vs Hazelcast OpenSource; again not sure if they have tuned Hazelcast in right way or not.

### 2. Community

They really lack here: https://redis.io/community

If you aren't on slack or gitter then I don't think there is any community.

### Operations

According to docs it should be simple; but don't have any experience for running it on large scales.

Couple of operators are available; which is good thing!

### Development

Given (Java) Spring it just abstracts away the under lying technology; so, basically no difference for developers.

## Synching multiple clusters

Both Redis and Hazelcast have paid products for synching; but other following OpenSource solutions can be used to setup replication/synching:

- https://github.com/twitter/twemproxy
- https://github.com/Netflix/dynomite
