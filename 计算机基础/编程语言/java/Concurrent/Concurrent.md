从ReentrantLock来看AQS独占锁实现

获取锁:

	• compareAndSetState(0,1) 当前没有任何线程获取，直接获取锁
	• Acquire(1) 请求1个信号量的锁
		○ tryAcquire 交由子类自己实现
		• addWaiter 将当前线程封装为Node传入，加入双端队列尾部
		○ acquireQueued 当前Node开始排队
			○ shouldParkAfterFailedAcquire 判断当前节点的前一个节点的等待状态是什么，如果是SINGAL的话当前线程需要被park
			○ parkAndCheckInterrupt park当前线程
		
释放锁：

	○ Release
		○ tryRelease  交由子类自己实现
		○ unparkSuccessor unpark当前队列头的线程
		

从CountDown来看共享锁实现

	○ setState 初始化state
	○ releaseShared
		○ tryReleaseShared state - 1，如果为0的话返回true
		○ doReleaseShared  唤醒队列中所有等待的线程

Condition

	• Awit 将当前线程持有的Lock 释放掉，并放入条件队列
Singal 将当前线程从条件队列中取出，并放入lock的队列