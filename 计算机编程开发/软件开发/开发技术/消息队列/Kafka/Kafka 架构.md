[[Kafka 概述|Kafka]]

### Kafka 体系结构

- **Producer**：生产者，负责创建消息，发送到 Kafka 中
- **Consumer**：消费者，负责接收消费 Kafka 中的消息
- **Broker**：服务代理节点，可以简单的看作是一个 Kafka 服务实例，负责处理客户端请求和消息持久化，一个或多个 Broker 组成一个 Kafka 集群

![](https://cdn.nlark.com/yuque/0/2023/png/26094224/1684377804803-3c105af4-09ad-4c0a-8452-650501d36ad1.png)

### 主题（Topic）与分区（Partition）

- **Topic：** 生产者将消息发送到特定的主题，消费者通过订阅特定的主题来消费消息
- **Partition：**

- 一个 Partition 属于属于单个 Topic，一个 Topic 可以有多个 Partition，同一个 Topic 下的 Partition 可以分布在不同的 Broker 上，即一个 Tpoic 可以横跨多个 Broker
- Partition 在存储层可以看作一个可追加的 Log 文件，消息被追加到日志文件时会分配一个特定的 offset 偏移量，offset 是消息在 Partition 中的唯一标识，通过 offset 保证消息在分区内的顺序性，但 offset 不能跨越分区

通过设置多分区实现水平扩展，增加并发系统量

### 分区多副本（Replica）机制

- 同一分区不同的副本保存相同的消息，副本之间是一主多从关系，leader 副本负责处理读写请求，follower 副本只负责同步 leader 副本数据
- 副本处于不同的 Broker 中，当 leader 副本出现故障时，从 follower 副本（ISR）中重新选举新的 leader 副本对外提供服务
- 副本相关概念：

- AR（Assigned Replicas）：分区中所有的副本
- ISR（In- Sync Replicas）：所有与 leader 副本保持一定同步程度的副本
- OSR（Out-of-Sync Replicas）：与 leader 副本同步滞后过多的副本
- leader 副本负责维护 ISR 集合与 OSR 集合

- HW 与 LEO：

- LEO（Log End Offset）：标识当前日志文件下一条待写入消息的 offset
- HW（High Watermark）：ISR 集合中最小的 LEO 即为分区的 HW，消费者只能消费 HW 之前的消息