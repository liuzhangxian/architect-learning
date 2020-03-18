## 结论

1. eureka实现服务注册与发现功能。


## 详细解释

<font color=blue>3月16日</font>
#### 服务注册与发现是什么？
1. 注册：有一个注册中心（如：eureka集群），维护了一个服务清单。每个服务单元启动后，将自己的ip、端口、协议提交到注册中心。注册中心以心跳的方式检测服务单元是否可用，从而实现清单的动态维护。
2. 发现：A启动后定时的到注册中心拉取服务清单，要调用其他服务时，从清单中获取ip、端口、协议。


#### 为什么要服务注册与发现？
1. 无需维护服务清单：A服务调用B服务，为了实现高可用，A需要维护B的服务清单。大量的微服务，会面临大量的服务清单维护，带来大量人力成本，同时容易发生错误。
2. 服务清单的动态变化：A调用B，如果B的一个服务不可用了，需要人工修改清单。注册中心可通过健康检查可动态删除不可用服务的清单。新服务启动，注册到服务中心，可动态添加新的清单。


#### 什么是eureka的服务端和客户端？
1. 服务端：eureka服务集群，也就是注册中心，维护服务清单。通过集群来实现高可用性，集群中的各个eureka服务异步进行清单的同步。通过心跳进行健康检查，动态删除清单。
2. 客户端：主要负责服务的注册与发现，一般即是服务提供者，也是服务消费者。嵌入在应用服务中，应用服务启动后注册并且拉取注册中心清单。

<font color=blue>3月17日</font>
#### 服务治理机制
服务提供者<font color=red>注册服务</font>，并定时通过心跳保持<font color=red>服务续约</font>，服务正常关闭会发送rest请求给注册中心来<font color=red>服务下线</font>。服务消费者定时<font color=red>拉取服务</font>，使用拉取到的服务清单来进行<font color=red>服务调用</font>。注册中心节点间<font color=red>同步服务</font>清单,定时<font color=red>剔除失效</font>未续约的服务，心跳失败的比例在15分钟内低于85%还可能触发<font color=red>自我保护</font>。

#### 源码分析
1. 客户端用@EnableDiscoveryClient注解来开启DiscoveryClient实例，这个实例引用了EurekaClient的实例类DiscoveryClient类。
2. 服务端用@EnableEurekaServer注解来启动
3. @EnableDiscoveryClient和@EnableEurekaCilent区别：注册中心是eureka则两者都能用，如果是其他注册中心，只能用前者。前者实现基于spring-cloud-commons,后者实现基于spring-cloud-netflix。
4. spring boot的加载，可以从META-INFO/spring.factories文件着手。

#### @EnableDiscoveryClient注解原理是什么？
1.

<font color=blue>3月18日</font>

#### nacos和eureka的区别是什么？
1. eureka已经不进行迭代升级，官方停止维护。nacos还在持续升级中。
2. eureka遵循AP（可用+分区容错）原则，牺牲了一定的一致性。nacos可以选择CP（一致性+分区容错）原则或AP（可用+分区容错）原则
3. eureka使用http协议，nacos可以选择http/动态DNS/UDP协议
4. eureka只支持spring cloud集成，nacos支持spring cloud、k8s、dubbo

#### 什么是CAP理论？
1. 分布式框架中的重要理论，即：CAP不可能都取，只能取其中两个。一般是选择牺牲一致性，通过"最终一致"来解决一致性问题。
2. 一致性（Consistency）（所有节点在同一时间具有相同的数据）
3. 可用性（Availability）（保证每个请求，不管成功或失败都有响应）
4. 分区容错（Partition tolerance）（系统中任意信息丢失或失败不会影响系统继续运作）