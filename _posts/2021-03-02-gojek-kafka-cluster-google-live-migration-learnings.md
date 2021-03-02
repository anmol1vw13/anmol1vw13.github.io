---
layout: post
title: Improving Kafka streams reliability by leveraging default.api.timeout.ms 
date: '2021-03-02T12:00:00.000-08:00'
author: Anmol Vijaywargiya
tags:
- tech
- kafka
- kafka_cluster
- zookeeper
- gojek
---

Recently, at Gojek, we faced a very peculiar issue related to kafka.

A lot of customers reported seeing untimely failures on producers and lag on consumers.
When people dug deep and compared all the times this had happened over the past few weeks, 
they figured out, migration of kafka broker(s) done by google to be the common event.

This wasn't a normal migration but a live migration (https://cloud.google.com/compute/docs/instances/live-migration), 
wherein the virtual machine instance is kept running and it is live migrated to another host on the same zone 
instead of requiring the VM to be rebooted.

Produce failures were of the following type

```
failed to publish message with exception with topic xxxx
java.util.concurrent.ExecutionException: 
org.apache.kafka.common.errors.NotLeaderForPartitionException: This server is not the leader for that topic-partition."
```

While, there were two types of consumers facing the lag
1. Ones which auto-recovered after a while.
1. Ones, the streams of which shut down and a recovery meant restarting the service.

Why and how a live migration would be the culprit was still an unknown!











