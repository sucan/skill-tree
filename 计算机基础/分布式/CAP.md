# CAP 定理

## 简介

`Eric Brewer` 在2000年提出，于2002年被证明。

> The CAP theorem is also called Brewer’s Theorem, because it was first advanced by Professor Eric A. Brewer during a talk he gave on distributed computing in 2000. Two years later, MIT professors Seth Gilbert and Nancy Lynch published a proof of “Brewer’s Conjecture

核心思想为分布式系统有三个关键指标：

1. Consistency
2. Availability
3. Partition tolerance

CAP定理认为这三个指标不可能同时实现

## Consistency

Consistency 一致性：

在同一时间所有客户端看到的数据都是同一份，意味着一旦任意一个节点的数据产生了变化那么在声明这个变化已经成功之前，必须要将数据转发或者复制到其他所有节点

## Availability

Availability 可用性：

任意一个客户端的任意一个请求都能够得到响应，即使一个或者多个节点已经损坏

## Partition tolerance

Partition tolerance 分区容错：

哪怕出现任意多个节点之间的通信断开或者延迟，集群依然会继续工作

## 参考资料

1. [cap-theorem](https://www.ibm.com/cloud/learn/cap-theorem)