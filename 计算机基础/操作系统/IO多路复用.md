# IO多路复用

## 是什么

IO多路复用(I/O multiplexing)是一种IO模型，出自[《unix network programming》第六章](https://www.cs.huji.ac.il/course/2004/com1/Exercises/Ex4/I.O.models.pdf)

IO多路复用的底层原理在于linux进程的睡眠和唤醒机制，当没有io事件发生时进程被休眠当有io事件发生时进程被唤醒并返回响应事件，目前linux上用于实现这个机制的api有多个，其中select 和 poll最早出现，epoll使用范围最广

## 实现API

### Select

        int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);

fd_set 的结构

    #define __FD_SETSIZE	1024
    typedef struct {
	    unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
    } __kernel_fd_set;



select 的主要实现逻辑如下：

1. 遍历所有fd，检查其是否有事件发生，例如可读可写等，并将当前进程放入每个fd的等待队列中
2. 遍历完所有的fd之后如果有事件发生则返回，如果没有事件返回则进程进入阻塞状态等待被唤醒
3. 进程被唤醒后继续执行步骤1


### Poll

    unsigned int (*poll)(struct file * fp, struct poll_table_struct * table)



### ePoll



## 其他知识点

1. 网卡接收到数据时会向cpu发起一个中断信号，CPU执行中断程序处理数据。
2. socket本质上也是一个文件有文件描述符,socket中有发送缓存区，接收缓存区，和等待队列

   
   