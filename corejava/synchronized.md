#### 2020年5月14日 早
思君如满月，夜夜减清辉。想去帮你修吹风机，修会响的床，想按摩...

#### synchronized的特性
1. 原子性：一个线程执行synchronized代码块内的所有的操作，不会被打断，其他线程不能执行这块代码。volatile不具备原子性。
2. 可见性：保证一个线程对一个变量的修改，其他线程及时可见。
3. 有序性
4. 可重入性：子类同步方法调用父类同步方法，可重入，不会死锁。

#### synchronized可以修饰哪些内容
1. 修饰方法，对对象加锁
2. 静态方法，对类对象加锁
3. 代码块，对对象实例加锁
4. 代码块，对类对象加锁

#### synchronized底层原理是什么？
1. 代码块用指令monitorenter，monitorexist实现；修饰方法用ACC_SYNCHRONIZED标识，告知虚拟机进入方法前先获得锁；
2. 两次monitorexit是防止异常，服务释放锁。
2. 通过对象头关联的monitor实现。
3. 对象布局，对象头的Mark Word存储了关联的monitor地址和对象的锁信息， 如锁状态：无锁，偏向锁，轻量级锁，重量级锁。

#### monitor监视对象如何实现？
1. monitor对象由C++实现，每个对象存在一个monitor对象与之关联，随对象一起创建、销毁
2. monitor对象中的_WaitSet队列：存放调用wait方法后的线程，等待被唤醒。
3. monitor对象中的_EntryList队列：等待锁的线程，进入_EntryList中
4. monitor对象中的_Owner: 指向持有monitor对象的线程
5. monitor对象中的count：持有monitor的线程数
5. monitor本身不能实现互斥，借助操作系统的mutex lock实现
6. 因为monitor存在于对象头中，所以notify、wait方法要存在于Object中。

#### monitor是如何实现线程安全的？

#### 等待状态的线程是如何进入等待状态的？
线程内调用wait方法，会释放锁，monitor的count减1，线程不在持有monitor，想成进入monitor的waitSet队列中。

#### synchronized锁的优化：核心优化点？如何优化？
1. 核心优化点：锁升级
2. 无锁
3. 偏向锁：一次cas操作。
```
1. 检查对象头mark work是否是偏向锁状态
2. 若为偏向锁状态，比较mark work中的线程id是否为当前线程id，如果是，执行步骤5，否则执行步骤3
3. mark work中的线程id与当前线程id不一致，通过CAS操作竞争锁，成功则把mark work中线程id换成当前线程id，后执行5，否则执行4
4. cas竞争失败，线程挂起，偏向锁升级为轻量级锁。
5. 执行同步代码块
```
4. 轻量级锁：自旋锁，自适应自旋锁。多次cas操作。
5. 重量级锁：monitor实现，借助操作系统mutex lock实现，需要用户态和内核态的切换，开销大。