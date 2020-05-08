#### 2020年5月8日 晚
给你科普一下辣的级别：微辣、中辣、特辣、变态辣、我想你辣。

#### rpc调用过程
1. client以本地调用的方式，调用client stub
2. client stub把参数的方法封装成消息
3. cient stub获取远程服务地址，发送消息
3. server stub收到消息，消息解码出方法和参数
4. server stub调用本地服务，获取返回结果
5. server stub封装返回结果，返回给client stub
6. client stub将结果反序列化成对象，返回给client
7. client 得到到返回结果

#### rpc实现
1. 封装：代理模式（动态代理）封装2-6步
2. 编码：接口名，方法名，参数类型，参数值，超时时间，requestID；返回值，返回code，requestID
3. 通信：NIO，基于netty，mina，java nio。
4. requestID：保证request和response匹配。当远程调用后ConcurrentHashMap里面put(requestID, callback)，当前线程wait，当得到返回结果后，从map中过去callback继续执行。


#### 发布服务
eureka，zookeeper等服务注册工具。

#### feign的实现