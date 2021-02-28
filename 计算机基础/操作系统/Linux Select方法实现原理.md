
## Select 方法实现原理

> select的内核实现路径为:linux/fs/select.c

### Select方法作用

实现IO多路复用的一个系统调用，可以用来监听多个socket上发生的IO事件

### 实现

内核入口：
   
    SYSCALL_DEFINE5(select, int, n, fd_set __user *, inp, fd_set __user *, outp,
		fd_set __user *, exp, struct __kernel_old_timeval __user *, tvp)
    {
	    return kern_select(n, inp, outp, exp, tvp);
    }

核心实现函数为`do_select`,调用路径：kern_select()->core_sys_select()->do_select()

简化后的源码如下：

    static int do_select(int n, fd_set_bits *fds, struct timespec64 *end_time)
    {
	......
	struct poll_wqueues table;
	poll_table *wait;
	......
	//1. 初始化poll_wqueues
	poll_initwait(&table);
	wait = &table.pt;
	......
	for (;;) {
	......
		for (i = 0; i < n; ++rinp, ++routp, ++rexp) {
			
			__poll_t mask;

			for (j = 0; j < BITS_PER_LONG; ++j, ++i, bit <<= 1) {
				struct fd f;
				......
				f = fdget(i);
				if (f.file) {
					......
					//2. 调用对应驱动程序的poll方法实现
					mask = vfs_poll(f.file, wait);
					......

				}
				......
			}
		}
		......
		//3.休眠当前进程
		if (!poll_schedule_timeout(&table, TASK_INTERRUPTIBLE,
					   to, slack))
			timed_out = 1;
	}
	......
	//4.清空poll_wqueues
	poll_freewait(&table);
	......
    }

1. 首先是初始化poll_wqueues,将当前进程赋值至poll_wqueues中的polling_task变量，将poll_wqueues.poll_table中的poll_queue_proc方法置为__pollwait
2. 这一步会调用socket中的file_operations.poll方法，该方法的实现是由具体的驱动程序去实现的，我们以`scull`设备实现的方法为例，其流程如下：
   1. 调用内核的poll_wait()方法，传入的入参分别为：设备自身，设备的等待队列，第一步中的poll_wqueues.poll_table
      1. 内核中的poll_wait实现如下
   
                static inline void poll_wait(struct file * filp, wait_queue_head_t * wait_address, poll_table *p)
                {
	                if (p && p->_qproc && wait_address)
		            p->_qproc(filp, wait_address, p);
                }
            其中_qproc方法已经在第一步被赋值为了__pollwait
       2.  __pollwait 的实现如下

                static void __pollwait(struct file *filp, wait_queue_head_t *wait_address,
				poll_table *p)
                {
                	//通过poll_table获取到其对应的poll_wqueues
	                struct poll_wqueues *pwq = container_of(p, struct poll_wqueues, pt);
	                //poll_wqueues中有一个存储poll_table_entry的数组，这个是返回数据中当前未使用的第一个存储地址
	                struct poll_table_entry *entry = poll_get_entry(pwq);
	                if (!entry)
		                return;
	                entry->filp = get_file(filp);
	                entry->wait_address = wait_address;
	                entry->key = p->_key;
	                init_waitqueue_func_entry(&entry->wait, pollwake);
	                entry->wait.private = pwq;
	                //将当前entry加入到等待队列中
	                add_wait_queue(wait_address, &entry->wait);
                } 
            其中pollwake方法是用于唤醒进程时调用的方法
3. 调用schedule方法将当前进程休眠
   1. 当IO设备接收到到IO事件时，会去唤醒自身阻塞队列中的所有进程，唤醒进程调用的就是前面提到的pollwake方法
4. 进程被唤醒之后，清空poll_wqueues,结束一次select调用

## 其他

### select方式调用涉及到的数据结构

![sad](/img/select/select数据结构.png)

### select方法核心类图
![sad](/img/select/select方法主要类图.png)

### container_of(ptr,type,member) 函数的作用

已知结构体type的成员member的地址ptr，求解结构体type的起始地址