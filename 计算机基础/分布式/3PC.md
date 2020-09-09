# Three phase commited

## 简介

三阶段提交协议，用于解决分布式数据库中的事务问题，保证ACID属性，是两阶段提交的优化版本

具体的三个阶段如下：

1. CanCommit
2. PreCommit
3. doCommit

三阶段提交主要是为了解决两阶段提交协议在某些场景下可能出现的阻塞问题，主要区别在于最后的commit阶段，两阶段提交是直接发送持久化命令给所有参与者，在命令没有到达之前所有参与者都不知道自己是否可以提交，而三阶段提交的commit阶段，所有参与者在没有接收到持久化命令之前都知道协调者更倾向于提交，所以不会导致参与者阻塞

所以三阶段协议是在不会出现网络分区的前提下使用的协议，因为其开销原因，并没有被广泛使用

## CanCommit

协调者询问所有参与者，是否可以执行事务
参与者判断自身状态，给于回复

## PreCommit

如果所有参与者都可以执行事务，那么协调者继续发送预提交命令给所有参与者，否则中断事务。

参与者接收到预执行命令之后，开始执行事务,但不提交(同两阶段提交的预执行阶段)，执行成功后返回结果

## doCommit

当所有参与者都返回成功之后，协调者发送提交事务命令，否则发送中断事务命令

但参与者接收到提交事务命令，或者超时之后，执行commit操作


## 存在的问题

除了能解决因为协调者宕机导致的阻塞问题之外，依然存在在发生网络故障时数据不一致问题



## 参考资料

1. [Three Phase Commit Protocol](https://www.geeksforgeeks.org/three-phase-commit-protocol/#:~:text=Three-Phase%20Commit%20%283PC%29%20Protocol%20is%20an%20extension%20of,fail%2C%20where%20we%20assume%20%E2%80%98k%E2%80%99%20is%20predetermined%20number.)
2. [关于分布式事务、两阶段提交协议、三阶提交协议](http://www.hollischuang.com/archives/681)