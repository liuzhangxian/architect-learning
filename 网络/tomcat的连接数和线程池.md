
#### tomcat nio 原理
tomcat nio 实现是基于JDK的java.io包。

1. http请求过程：Acceptor(accept queue，LimitLatch) -> Poller events queue -> Poller(selector) -> Worker(线程次)
2. acceptor：负责接收socket连接，默认一个线程，将socketChannel封装成event事件放到poller event queue中。
2. LimitLatch: 连接控制器，负责维护连接数，达到maxConnections后拒绝连接。
3. accept queue: 当连接达到最大连接数，新连接放到这个队列中。
4. Poller events queue: 请求事件队列，nio核心。
5. Poller: 负责轮询，线程数默认是cpu核数。维护了selector对象，poller从event queue中去事件注册到selector中。
6. selector: poller维护的对象，负责遍历读数据的socket拿给worker执行。
7. selectorPool: selector的线程池。
7. Worker: 从socket中读取request，封装，分派到相应的servlet完成请求，将servlet的response中写回socket。

#### 三个参数：maxThreads、maxConnections、acceptCount
1. maxThreads: 处理请求的工作线程的最大线程数。默认200。
2. maxConnections：tomcat在任意时刻接收和处理的最大连接数。NIO默认10000.
3. acceptCount: accept队列长度。默认100。当连接数达到maxConnections时，新进来的请求放到accept队列中，超过acceptCount时，后面的请求会被拒绝。

#### 查看tomcat当前状态
1. 查看连接数：netstat –nat | grep {端口号}
2. 查看进程线程状态：stack -l pid