[[Kafka 概述|Kafka]]

## 消费者与消费者组

多个消费者实例共同组成一个消费者组，通过 Consumer Group 整体订阅某个 Topic，Consumer Group 中的Consumer 共同消费 Topic 内的所有分区

Kafka 通过 Group Id 唯一标识 Consumer Group

Consumer Group 订阅 Topic 后，Topic 中的每个 Partition 只能被一个 Consumer Group 中的一个 Consumer 所消费