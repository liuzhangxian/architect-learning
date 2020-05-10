#### 2020年5月10日 下午

#### 需要具备的知识
1. 线程状态
2. 锁的实现原理
3. 线程栈

#### 命令使用
1. jstack -l pid： 会输出当时进程的所有线程的基本信息、调用栈等

#### thread dump信息分析（一般多个dump对比分析）
1. 线程状态（java.lang.Thread.State: WAITING）：RUNNABLE，线程处于执行中；BLOCKED，线程被阻塞；WAITING，线程正在等待
2. 锁状态：
locked <0x000000076bf62208>说明该线程对地址为0x000000076bf62208对象进行了加锁；
waiting to lock <0x000000076bf62208> 说明该线程在等待地址为0x000000076bf62208对象上的锁；
waiting for monitor entry [0x000000001e21f000]说明线程1是通过synchronized关键字进入了监视器的临界区，并处于"Entry Set"队列，等待monitor

#### 在线thread dump文件分析工具
https://fastthread.io/