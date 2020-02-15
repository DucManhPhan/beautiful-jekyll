---
layout: post
title: Consumer in Kafka
bigimg: /img/path.jpg
tags: [Kafka]
---



<br>

## Table of contents





<br>

## The definition of Kafka Consumer





<br>

## How Kafka Consumer works


Every time a consumer polls new messages, a heart-beat is sent to the group co-ordinator (A group co-ordinator is the first broker reporting its availability to Zookeeper). If the heart-beat is not received by the group coordinator then it assumes that the consumer is down and re-assigns its partition to other available consumer – process called rebalancing.

After a consumer reads and processes a message it needs to commit the offset of that message. This keeps a consumer and last read message in sync.

If a leader partition goes down, then another in-sync replicated partition becomes a leader and rest of the partitions start keeping up with the new leader partition.

<br>

## Source code Java for Kafka consumer






<br>

## The meaning of parameters in consumer client




- Why we need group.id when configuring some fields for creating consumer



<br>

## Coordinator in Kafka

[https://jaceklaskowski.gitbooks.io/apache-kafka/kafka-consumer-internals-ConsumerCoordinator.html](https://jaceklaskowski.gitbooks.io/apache-kafka/kafka-consumer-internals-ConsumerCoordinator.html)


<br>

## Rebalancing in Kafka

1. When rebalacing between consumers happens

    Whenever a consumer enters or leaves a consumer group, the brokers rebalance the partitions across consumers, meaning Kafka handles load balancing with respect to the number of partitions per application instance for us. This is great—it's a major feature of Kafka. We use this default on nearly all our services.

    When a rebalance happens, all consumers drop their partitions and are reassigned new ones. If you have an application that has state associated with the consumed data, such as our aggregator service, we need to drop that state and start fresh with data from the new partition.

    However, if dropping state isn’t an option, an alternative is to not use a consumer group and instead use the Kafka API to statically assign partitions, which does not trigger rebalances. Of course, in that case, you must balance the partitions yourself and also make sure that all partitions are consumed.


2. The affections of rebalancing in Kafka


<br>

## Heartbeat thread in Kafka




<br>

## Thread-safe with consumer
1. One Consumer per Thread

According to [https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/ch04.html](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/ch04.html), we have:

```
You can't have multiple consumers that belong to the same group in one thread and you can't have multiple threads safely use the same consumer. One consumer per thread is the rule. To run multiple consumers in the same group in one application, you will need to run each in its own thread. It is useful to wrap the consumer logic in its own object and then use Java's ExecutorService to start multiple threads each with its own consumer.
```

By using one thread per consumer, we have some benefits and drawbacks:
- Benefits

    - It is the easiest to implement.
    - It is often the fastest as no inter-thread co-ordination is needed.
    - It makes in-order processing on a per-partition basis very easy to implement (each thread just processes messages in the order it receives them).

- Drawbacks

    - More consumers means more TCP connections to the cluster (one per thread). In general Kafka handles connections very efficiently so this is generally a small cost.
    - Multiple consumers means more requests being sent to the server and slightly less batching of data which can cause some drop in I/O throughput.The number of total threads across all processes will be limited by the total number of partitions.

2. Decouple consumption and processing

    If we do not want to perform one thread per consumer, we can use an alternative way that is to have one or more consumer threads that do all data consumption and hands off ConsumerRecords instances to a blocking queue consumed by a pool of processor threads that actually handle the record processing.

    - Benefits:

        - This option allows independently scaling the number of consumers and processors. This makes it possible to have a single consumer that feeds many processor threads, avoiding any limitation on partitions.

    - Drawbacks

        - Guaranteeing order across the processors requires particular care as the threads will execute independently an earlier chunk of data may actually be processed after a later chunk of data just due to the luck of thread execution timing. For processing that has no ordering requirements this is not a problem.

        - Manually committing the position becomes harder as it requires that all threads co-ordinate to ensure that processing is complete for that partition.

    There are many possible variations on this approach. For example each processor thread can have its own queue, and the consumer threads can hash into these queues using the TopicPartition to ensure in-order consumption and simplify commit.

<br>

## Some problems when running Kafka
1. ```attempt to heartbeat failed since group is rebalancing```

[https://stackoverflow.com/questions/40162370/heartbeat-failed-for-group-because-its-rebalancing](https://stackoverflow.com/questions/40162370/heartbeat-failed-for-group-because-its-rebalancing)


<br>

## Wrapping up







<br>

Refer:

[https://ohioedge.com/2018/05/17/kafka-a-simple-explanation/](https://ohioedge.com/2018/05/17/kafka-a-simple-explanation/)

[https://blog.newrelic.com/engineering/kafka-best-practices/](https://blog.newrelic.com/engineering/kafka-best-practices/)

[https://blog.newrelic.com/engineering/kafka-consumer-config-auto-commit-data-loss/](https://blog.newrelic.com/engineering/kafka-consumer-config-auto-commit-data-loss/)

[https://www.cloudkarafka.com/blog/2016-11-30-part1-kafka-for-beginners-what-is-apache-kafka.html](https://www.cloudkarafka.com/blog/2016-11-30-part1-kafka-for-beginners-what-is-apache-kafka.html)

<br>

**Hearbeat thread**

[http://javierholguera.com/2018/01/01/timeouts-in-kafka-clients-and-kafka-streams/](http://javierholguera.com/2018/01/01/timeouts-in-kafka-clients-and-kafka-streams/)

[https://stackoverflow.com/questions/39730126/difference-between-session-timeout-ms-and-max-poll-interval-ms-for-kafka-0-10-0/39759329#39759329](https://stackoverflow.com/questions/39730126/difference-between-session-timeout-ms-and-max-poll-interval-ms-for-kafka-0-10-0/39759329#39759329)

[https://devguli.com/heartbeat-thread-in-kafka-consumer/](https://devguli.com/heartbeat-thread-in-kafka-consumer/)

[https://cwiki.apache.org/confluence/display/KAFKA/KIP-62%3A+Allow+consumer+to+send+heartbeats+from+a+background+thread](https://cwiki.apache.org/confluence/display/KAFKA/KIP-62%3A+Allow+consumer+to+send+heartbeats+from+a+background+thread)

[https://chrzaszcz.dev/2019/06/kafka-heartbeat-thread/](https://chrzaszcz.dev/2019/06/kafka-heartbeat-thread/)

[https://stackoverflow.com/questions/40162370/heartbeat-failed-for-group-because-its-rebalancing](https://stackoverflow.com/questions/40162370/heartbeat-failed-for-group-because-its-rebalancing)

<br>

**Rebalancing**

[https://ibm-cloud-architecture.github.io/refarch-eda/kafka/arch/](https://ibm-cloud-architecture.github.io/refarch-eda/kafka/arch/)

[https://blog.newrelic.com/engineering/effective-strategies-kafka-topic-partitioning/](https://blog.newrelic.com/engineering/effective-strategies-kafka-topic-partitioning/)

[https://stackoverflow.com/questions/43991845/kafka10-1-heartbeat-interval-ms-session-timeout-ms-and-max-poll-interval-ms](https://stackoverflow.com/questions/43991845/kafka10-1-heartbeat-interval-ms-session-timeout-ms-and-max-poll-interval-ms)

[https://www.slideshare.net/ConfluentInc/everything-you-always-wanted-to-know-about-kafkas-rebalance-protocol-but-were-afraid-to-ask-matthias-j-sax-confluent-kafka-summit-london-2019](https://www.slideshare.net/ConfluentInc/everything-you-always-wanted-to-know-about-kafkas-rebalance-protocol-but-were-afraid-to-ask-matthias-j-sax-confluent-kafka-summit-london-2019)

<br>

**Multithreading with Kafka's consumer**

[https://howtoprogram.xyz/2016/05/29/create-multi-threaded-apache-kafka-consumer/](https://howtoprogram.xyz/2016/05/29/create-multi-threaded-apache-kafka-consumer/)

[https://www.reactiveprogramming.be/an-introduction-to-apache-kafka/](https://www.reactiveprogramming.be/an-introduction-to-apache-kafka/)

[https://www.reactiveprogramming.be/an-introduction-to-reactor-kafka/](https://www.reactiveprogramming.be/an-introduction-to-reactor-kafka/)

[https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html](https://kafka.apache.org/10/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)