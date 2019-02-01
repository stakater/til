# Redis vs Hazelcast

This document compares Redis with Hazelcast.

## Hazelcast

### Benchmark

Read these two:

- https://hazelcast.com/resources/benchmark-redis-vs-hazelcast/
- https://hazelcast.com/resources/whitepaper-redis-comparison/

### Community

Quite vibrant and awesome

https://hazelcast.org/get-involved/

Just sign up on gitter and then you can have chat with real users!

### Operations

Fairly simple; have used in really large scale projects and never had even single incident with Hazelcast.

### Development

Given (Java) Spring it just abstracts away the under lying technology; so, basically no difference for developers.

### Wins

- If you are Java shop then Hazelcast is unique in that it can be embedded in a Java host process, making it great for building stateful microservices without an external database dependency. It also has some other small differences, like the ability to get a call back from a key expiration. In a sense, it does less overall but the few things it does, it does them better. Especially if you're using Java.

## Redis



### Development

Given (Java) Spring it just abstracts away the under lying technology; so, basically no difference for developers.
