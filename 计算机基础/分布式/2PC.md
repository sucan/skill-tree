# Two phase commited

## 简介

两阶段提交，用于解决分布式数据库中的事务问题，保证ACID属性

这两阶段分别为：准备阶段和决策阶段

在两阶段提交中有两类角色，分别为一个协调者(coordinator)和多个参与者(participants)

## Prepare phase

协调者向所有参与者发送消息，要求参与者做好提交(commit)准备

参与者在收到消息之后判断自身能否做好提交事务的准备，如果能够得话就做好准备(我理解应该就和innodb写precommit的redolog是一个意思)，然后向协调者发送准备就绪的回复，如果不能执行，则返回拒绝执行


## Resolution phase

决策阶段有两种情况：

1. 所有参与者都返回准备就绪
2. 有参与者返回拒绝执行，或者有参与者超时未返回

针对第一种情况，协调者会向所有参与者发送提交事务的指令，等所有参与者都返回成功后，结束事务

针对第二章情况，协调者会向所有参与者发送取消事务的指令，等所有参与者都返回取消之后，结束事务

## 存在的问题

单点、同步阻塞、脑裂

1. 协调者只有一个，存在单点问题，在执行事务的过程中，所有参与者都会被加上锁，如果协调者挂了，就会出现一直被锁的问题。即使重新选举也没有办法解决
2. 数据可能不一致，例如协调者在发送commit命令的过程中宕机了



## 参考资料

1. [IBM](https://www.ibm.com/support/knowledgecenter/en/SSAL2T_9.1.0/com.ibm.cics.tx.doc/concepts/c_two_phz_commit_process.html)
2. [关于分布式事务、两阶段提交协议、三阶提交协议](http://www.hollischuang.com/archives/681)