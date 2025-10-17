---
title: seatunnel使用中遇到的各种问题
published: 2025-03-25
tags: ['java','seatunnel','大数据']
description: 'seatunnel local 模式多开、多版本共存、内存溢出'
category: '技术'
---

# seatunnel使用中遇到的各种问题

## local 模式内存溢出

修改seatunnel config 目录下的jvm_client_option

```text
# JVM Heap
-Xms256m
-Xmx1024m

# JVM Dump
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/seatunnel/dump/zeta-client
```

## local模式同时启动多个任务

修改修改seatunnel config目录下面的hazelcast-client.yaml

```yaml
hazelcast-client:
  cluster-name: seatunnel
  properties:
    hazelcast.logging.type: log4j2
  connection-strategy:
    connection-retry:
      cluster-connect-timeout-millis: 3000
  network:
    cluster-members:
      - localhost:5801
      - localhost:5802
      - localhost:5803
```

## local模式多版本共存

修改任意一个版本的hazelcast-client.yaml与hazelcast.yaml，并且确保 cluster-name 相同第一个端相同。

```yaml
hazelcast-client:
  cluster-name: seatunnel_2.3.9
  properties:
    hazelcast.logging.type: log4j2
  connection-strategy:
    connection-retry:
      cluster-connect-timeout-millis: 3000
  network:
    cluster-members:
      - localhost:5802
```

```yaml
hazelcast:
  cluster-name: seatunnel_2.3.9
  network:
    rest-api:
      enabled: true
      endpoint-groups:
        CLUSTER_WRITE:
          enabled: true
        DATA:
          enabled: true
    join:
      tcp-ip:
        enabled: true
        member-list:
          - localhost
    port:
      auto-increment: false
      port: 5802
  properties:
    hazelcast.invocation.max.retry.count: 20
    hazelcast.tcp.join.port.try.count: 30
    hazelcast.logging.type: log4j2
    hazelcast.operation.generic.thread.count: 50
    hazelcast.heartbeat.failuredetector.type: phi-accrual
    hazelcast.heartbeat.interval.seconds: 2
    hazelcast.max.no.heartbeat.seconds: 180
    hazelcast.heartbeat.phiaccrual.failuredetector.threshold: 10
    hazelcast.heartbeat.phiaccrual.failuredetector.sample.size: 200
    hazelcast.heartbeat.phiaccrual.failuredetector.min.std.dev.millis: 100
```
