## Linux 进程休眠和唤醒
> 涉及到这部分功能的源码实现大部分都在 include/linux/sched.h 中
## 进程

在linux内核中进程使用`task_struct`结构体来表示,其中有个字段为`state`,表示当前进程的状态，正在运行或者等cpu时间片的进程状态为**TASK_RUNNING**，在运行中的进程会被放到一个cpu维度全局唯一的运行队列中。

休眠状态的进程的state有两种，分别为：
  1. TASK_INTERRUPTIBLE（使用评率最多，一直睡眠，可以通过接收到信号量 || 某个条件为真时被唤醒）
   1. TASK_UNINTERRUPTIBLE（一直睡眠，直到某个条件为真时被唤醒）

### 休眠

    sleeping_task = current;//current是一个宏，可以拿到当前的task_struct对象
    set_current_state(TASK_INTERRUPTIBLE);
    schedule();
    func1();

1. 修改当前进程的task_struct结构体中的state变量值为**TASK_INTERRUPTIBLE** 
2. 调用`schdule(void)`方法使进程陷入休眠，如果schedule()是被一个状态为**TASK_RUNNING**的进程调度，那么schedule()将调度另外一个进程占用CPU；如果schedule()是被一个状态为TASK_INTERRUPTIBLE 或TASK_UNINTERRUPTIBLE 的进程调度，那么还有一个附加的步骤将被执行：当前执行的进程在另外一个进程被调度之前会被从运行队列中移出，这将导致正在运行的那个进程进入睡眠，因为它已经不在运行队列中了。


### 唤醒

    wake_up_process(sleeping_task); 

1. 获取到要唤醒的进程的`task_struct`对象
2. 调用`wake_up_process(struct task_struct *p)`方法(最终调用的是`try_to_wake_up`)，该方法会修改进程的状态为**TASK_RUNNING**，并将该进程插入运行队列（cpu维度，每个cpu独立维护一个运行队列）中。