# ScheduledThreadPool



## 1. 使用方式

```java
1.schedule(Runnable command,long delay,TimeUnit unit);
2.scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);
3.scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit) ;
```

1. 延迟 delay时间后，执行线程，只执行一次
2. 延迟 initialDelay时间后，执行线程，每 period时间执行一次（**scheduleAtFixedRate：固定周期**）
3. 延迟 initialDelay时间后，执行线程，前一次执行完后，相隔 delay时间再执行（**scheduleWithFixedDelay：固定间隔**）



## 2. DelayedWorkQueue

ScheduledThreadPool 使用 DelayedWorkQueue。

堆（heap）也被称为优先队列。

在堆底插入元素，在堆顶取出元素。二叉树的衍生，有最小堆最大堆的两个概念，将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。常见的堆有二叉堆、斐波那契堆等。



ReentrantLock 保证线程安全

Condition available 控制线程 await，signal

heapIndex字段，以记录其在最小堆中的位置。



## 3. Leader

```java
/**
 * Thread designated to wait for the task at the head of the
 * queue.  This variant of the Leader-Follower pattern
 * (http://www.cs.wustl.edu/~schmidt/POSA/POSA2/) serves to
 * minimize unnecessary timed waiting.  When a thread becomes
 * the leader, it waits only for the next delay to elapse, but
 * other threads await indefinitely.  The leader thread must
 * signal some other thread before returning from take() or
 * poll(...), unless some other thread becomes leader in the
 * interim.  Whenever the head of the queue is replaced with a
 * task with an earlier expiration time, the leader field is
 * invalidated by being reset to null, and some waiting
 * thread, but not necessarily the current leader, is
 * signalled.  So waiting threads must be prepared to acquire
 * and lose leadership while waiting.
 */
private Thread leader;
```



调用 take 获取任务时，只有 leader线程调 available.awaitNanos(delay)，等待任务delay时间后执行。

其他线程 await 等待，直到 leader 执行后 singal 唤醒。

抢锁->尝试获取队列的任务-> 尝试成为leader->等待delay后执行。





若新增元素位于堆顶，需安排定期调度或立即执行该任务的leader。

offer方法在新增元素应率先调度时，清空leader，并signal唤醒某个等待线程，继续take方法中的循环:抢锁->尝试获取队列的任务-> 尝试成为leader->等待delay后执行。





参考资料：

https://mp.weixin.qq.com/s/0u75TWniU-bwa8XdDeprIA