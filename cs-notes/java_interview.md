## 大厂面试题

### 一. JUC 多线程及高并发

#### 1. 请谈谈你对 volatile 的理解

> 1）volatile 是 Java 虚拟机提供的<span style="color:red">轻量级</span>的同步机制(乞丐版的synchronized)
>
> > - 保证可见性。某线程修改了变量的值，其他线程要可以立即知道最新值，其他线程中该变量的值作废（不是最新的）
> >
> > - <span style="color:red">不保证原子性</span>。被加塞了，可能出现写覆盖的情况，number++在多线程环境下是不安全的（字节码有好几条）。
> >   
> >   - 如何解决原子性？
> >     - 加 sync（杀鸡不应用牛刀）
> >     - 直接使用 juc 下的 AtomicInteger 类
> >   
> > - 禁止指令重排(有序性)
> >
> >   - 计算机在执行程序时，为了提高性能，编译器和处理器常常会对<span style="color:red">指令做重排</span>，一般分一下三种
> >
> >     ```mermaid
> >     graph LR
> >     A[源代码]-->B[编译器优化重排]-->C[指令并行的重排]-->D[内存系统的重排]-->E[最终执行的指令]
> >     ```
> >     
> >   - 单线程环境里面确保程序最终执行结果和代码顺序执行结果一致
> >   
> >   - 处理器在进行重排序时必须要考虑指令之间的<span style="color:red">数据依赖性</span>。
> >   
> >   - 多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能够保证一致性是无法确定的，结果无法预测
> >   
> >   ![my02](![](https://raw.githubusercontent.com/kokutou187/notes/master/images/my02.png))
> >   
> >   ![my03](![](https://raw.githubusercontent.com/kokutou187/notes/master/images/my03.png))
>
> 2）JMM（Java Memory Model，Java内存模型） 你谈谈
>
> > JMM 本身并<span style="color:red">不真实存在</span>，它描述的是一组规则或规范，该规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。
> >
> > JMM 关于同步的规定：
> >
> > > 1. 线程解锁前，必须把共享变量的值刷新回主内存
> > > 2. 线程加锁前，必须读取主内存的最新值到自己的工作内存
> > > 3. 加锁解锁是同一把锁
> >
> > 由于 JVM 运行程序的实体是线程，而每个线程创建时 JVM 都会为其创建一个<span style="color:red">工作内存（有些地方称为栈空间，指的应该是对应某个或某几个线程的栈帧中的OS（操作数栈）,以及操作数栈进行运算完以后会写入的LV局部变量表，然后主内存指的应该是堆或者方法区。）</span>，工作内存是每个线程的私有数据区域 ，而 Java 内存模型中规定所有变量都存储在<span style="color:red">主内存</span>，所有线程都可以访问，<span style="color:red">但线程对变量的操作（读取赋值等）必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存</span>，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的<span style="color:blue">变量副本拷贝，因此不同的线程间无法访问对方的工作内存</span>，线程间的通信（传值）必须通过主内存来完成，其简要访问过程如下图：
> >
> > <img src="![my01](https://raw.githubusercontent.com/kokutou187/notes/master/images/my01.png)" alt="my01" style="zoom:67%;" />
> >
> > JMM 的特性：
> >
> > - 可见性
> > - 原子性
> > - VolatileDemo 代码演示可见性 + 原子性代码
> > - 有序性(禁止指令重排)
>
> 3）你在哪些地方用到过 volatile？
>
> > - 单例模式 DCL（Double Check Lock双端检锁机制） 代码，即在同步代码块(锁)的前后都加判断来检查
> >
> >   ```java
> >   public static SingletonDemo getInstance() {
> >       private volatile static SingletonDemo instance;
> >       
> >      //DCL（Double Check Lock双端检锁机制）,
> >      if (instance == null) {
> >          synchronized (SingletonDemo.class) {
> >              if (instance == null) {
> >                  instance = new SingletonDemo();
> >              }
> >           }
> >       }
> >       return instance;
> >   }
> >   ```
> >
> >   - DCL机制不一定线程安全，原因是有指令重排的存在，加入 volatile 可以禁止指令重排。
> >
> >     > 原因在于某一个线程执行到第一次检测，读取到的 instance 不为 null 时，instance 的引用对象<span style="color:red">可能没有完成初始化</span>。instance = new SingletonDemo();  可以分为以下3步完成（伪代码）
> >     >
> >     > memory  =  allocate()  //1. 分配对象内存空间
> >     >
> >     > instance (memory)	//2. 初始化对象
> >     >
> >     > instance = memory 	//3. 设置instance指向刚分配的内存地
> >     >
> >     > ​											址，此时instance != null
> >     >
> >     > 步骤2和3 <span style="color:red">不存在数据依赖关系</span>，而且无论重拍前还是重排后程序的执行结果在单线程中并没有改变，因此这种重排优化是允许的。
> >     >
> >     > memory = allocate() 	//1. 分配对象内存空间
> >     >
> >     > instance = memory 	//3. 设置instance指向刚分配的内存地
> >     >
> >     > ​		址，此时instance != null, <span style="color:blue">但是对象还没有初始化完成!</span>。
> >     >
> >     > instance(memory)		//2. 初始化对象
> >     >
> >     > 但是指令重排只会保证串行语义的执行的一致性（单线程），但并不会关心多线程间的语义一致性。
> >     >
> >     > <span style="color:red">所以当一条线程访问 instance 不为 null 时，由于 instance 实例未必已初始化完成，也就造成了线程安全问题</span>。
> >     >
> >     > <span style="color:red">所以需要在单例模式的的实例前加 volatile 禁止指令重排</span>。
> >
> > - 单例模式 volatile 分析

#### 2. CAS 你知道吗

##### 2.1. 比较并交换

- CAS ==> compareAndSet，
  - atomicInteger.compareAndSet(expected, update)：atomicInteger的初始值是工作内存中的值，然后 expected 表示期望主物理内存中的值是什么，如果和参数 expected 一致就把两个地方的内存中的数据都修改为 update 的值。

##### 2.2 CAS底层原理？如果知道，谈谈你对 UnSafe 的理解

- atomicInteger.getAndIncrement();

  ```java
  public final int getAndIncrement() {
      //this 表示当前对象，valueoffset表示内存地址偏移量，通过unsafe类获取
      return unsafe.getAndAddInt(this, valueOffset, 1);
  }
  
  //Unsafe.class，自旋锁
  public final int getAndAddInt(Object var1, long var2, int var4) {
      int var5;
      do {
          //var1 = this, var2 = valueoffset，
          //将物理内存中的数据拷贝到工作内存中(var5)
          var5 = this.getIntVolatile(var1, var2);
          //比较物理内存中的值和工作内存中的 var5一致与否？
          //一致就 var5 + var4，这里 var4指的是1，不一致会一直循环
      } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
      return var5;
  }
  ```

- Unsafe

  - Unsafe 是 CAS 的核心类，由于 Java 方法无法直接访问底层系统，需要通过本地(native)方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。<span style="color:red">Unsafe类存在于sun.misc包中</span>，其内部方法操作可以像C的指针一样直接操作内存，因为 Java 中 CAS 操作的执行依赖于 Unsafe 类的方法。也就是<span style="color:red">通过Unsafe类保证了原子性</span>。
  - <span style="color:red">注意 Unsafe 类中的所有方法都是 native 修饰的，也就是说 Unsafe 类中的方法都直接调用操作系统底层资源执行响应任务</span>。
  - 变量 value 用 volatile 修饰，保证了多线程之间的内存可见性

- CAS是什么

  - CompareAndSwap/Set，<span style="color:red">它是一条 CPU  原语</span>。
  - CAS 并发原语体现在 JAVA 语言中就是 sun.misc.Unsafe 类中的各个方法。调用 Unsafe 类中的 CAS 方法，JVM 会帮我们实现<span style="color:blue">CAS 汇编指令</span>。它实现了原子操作，<span style="color:red">并发原语的执行必须是连续的，在执行过程中不允许被中断，也就是说 CAS 是一条 CPU 的原子指令，不会造成所谓的数据不一致问题</span>，从而保证了原子性。
  - 比较当前工作内存中的值和主内存中的值，如果相同则执行规定操作，否则继续比较直到主内存和工作内存中的值一致为止。

##### 2.3 CAS缺点

> synchronized 减少了并发性，CAS增加了并发性，因为它允许多个线程同时修改。
>
> 1. 循环时间长开销很大
> 2. 只能保证一个共享变量的原子操作，对于多个共享变量的操作只能使用锁了。
> 3. 引出来 ABA 问题

#### 3. 原子类 AtomicInteger 的ABA问题谈谈？原子更新引用知道吗？

- ABA问题是怎么产生的？

  > CAS算法实现一个重要前提需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差中，类会导致数据的变化。
  >
  > 比如说一个线程one从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且线程 two 进行了一些操作将值变成了 B，然后线程 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操作发现内存中依然是 A，然后线程 one 操作成功。
  >
  > <span style="color:red">尽管线程one的 CAS 操作成功，但是不代表这个过程就是没有问题的</span>。

- 原子引用

  ```java
  AtomicReference<User> atomicReference = new AtomicReference<>();
  ```

- 时间戳原子引用

  - 解决 ABA 问题？？  原子引用 + 版本号（类似时间戳）

  ```java
  AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(初始值: 100, 初始版本号: 1);
  ```

  

#### 4. 我们知道 ArrayList 是线程不安全，请编码写一个不安全的案例并给出解决方案

- 参考 JUC 笔记中的第三点



#### 5. 公平锁/非公平锁/可重入锁/递归锁/自旋锁谈谈你的理解？请手写一个自旋锁

- 公平锁和非公平锁
  - 概念
    - 公平锁：多个线程按照申请的顺序来获取锁，非抢占
    - 非公平锁：多个线程获取锁的顺序并不是按照申请锁的顺序，按优先级来获取，抢占式。有可能造成反转或者饥饿现象
    - Java ReentrantLock 通过构造函数指定该锁是否是公平锁，<span style="color:blue">默认是非公平锁</span>。非公平锁的优点在于吞吐量比公平锁大。对于 Synchronized 而言，也是一种非公平锁。

- 可重入锁（又名递归锁）
  - 概念
    - 同一线程外层函数获得锁之后，内层函数仍然能获取该锁的代码，在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁，也就是说，<span style="color:red">线程可以进入任何一个它已经拥有的锁所同步着的代码块</span>。
    - ReentrantLock / Synchronized 就是一个典型的非公平可重入锁，
  - 作用
    - 避免死锁

- 自旋锁 (SpinLock)   --- SpinLockDemo
  - 概念
    - 尝试获取锁的线程不会立即阻塞，而是<span style="color:red">采用循环的方式去尝试获取锁</span>，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗 CPU。
- 独占锁（写锁） / 共享锁（读锁）/互斥锁    --- ReadWriteLockDemo
  - 概念
    - 独占锁：锁一次只能被一个线程所持有，对 ReentrantLock 和 Synchronized 而言都是独占锁
    - 共享锁：锁可以被多个线程所持有
    - 读锁的共享锁可保证并发读是非常高效的，读写，写读，写写的过程是互斥的。



#### 6. CountDownLatch/CyclicBarrier/Semaphore使用过吗？

- 参考 JUC 笔记



#### 7. 阻塞队列知道吗？

- 参考 JUC 笔记
- 多线程的 3.0 版本，不需要自己控制了，不用 Lock



#### 8. 线程池用过吗？ThreadPoolExecutor谈谈你的理解？

- 参考 JUC



#### 9. 线程池用过吗？生产上你如何设置合理参数

- 参考 JUC



#### 10. 死锁编码及定位分析

- jps -l 命令定位进程号
- jstack 线程ID 找到死锁查看





### 二. JVM + GC解析

- JVM 垃圾回收的时候如何确定垃圾？是否知道什么是 GC Roots

  - 引用计数算法
  - 可达性分析

- 你说你做过的 JVM 调优和参数配置，请问如何盘点查看 JVM 系统默认值
  - JVM 的参数类型

    - 标配参数

      > -version	-help	-showversion

    - x 参数

      > -Xint：解释执行
      > -Xcomp：第一次使用就编译成本地代码
      > -Xmixed：混合模式

    - <span style="color:red">xx 参数</span>：

      - Boolean类型

        > -XX:+/-某个属性值
        > +表示开启，-表示关闭
        >
        > jps  -l 找到进程ID     jinfo -flag 命令参数  进程ID，查看是否开启

      - KV设值类型

        > -XX:key=value

      - jinfo 举例，如何查看当前运行程序的配置

        > jinfo  -flags  进程ID

      - 题外话

        > -Xms 等价于 -XX:InitialHeapSize
        > -Xmx 等价于 -XX:MaxHeapSize
        > 所以这两个属于 XX参数

  - JVM 系统默认值

    - jps  -l   jinfo -flag  具体参数   java进程ID  /  jinfo  -flags  java进程ID
    - java  -XX:+PrintFlagsInitial：查看 JVM 初始化默认参数
    - java  -XX:+PrintFlagsFinal：查看修改更新的参数，打出来的参数是 := 的表示这个参数修改过
    - -XX:+PrintCommandLineFlags

- 你平时工作用过的 JVM 常用基本配置参数有哪些

  - -Xms：初始内存大小，默认物理内存的 1/64，等价于 -XX:InitialHeapSize
  - -Xmx：最大分配内存，默认为物理内存 1/4，等价于 -XX:MaxHeapSize
  - -Xss：设置单个线程栈的大小，一般默认为 512K~1024K，等价于 -XX:ThreadStackSize
  - -Xmn：设置年轻代大小
  - -XX:MetaspaceSize：设置元空间大小，元空间不在虚拟机中，而是使用本地内存
  - -XX:+PrintGCDetails：输出详细的 GC 收集日志信息
  - -XX:SurvivorRatio：设置新生代 eden 和 S0/S1 空间的比例，默认 -XX:SurvivorRatio=8，Eden:S0:S1=8:1:1。
  - -XX:NewRatio：配置年轻代与老年代在堆结构的占比，默认 -XX:NewRatio=2，新生代占1，老年代2，年轻代占整个堆的 1/3
  - -XX:MaxTenuringThreshold：设置垃圾最大年龄，默认为15，设置为0，则年轻对象不经过 Survivor 区，直接进入老年代

- 强引用、软引用、弱引用、虚引用分别是什么

  - 见 java 虚拟机笔记

- 请谈谈你对 OOM 的认识

  - java.lang.StackflowError，错误

  - java.lang.OutOfMemoryError:  java  heap  space，错误

  - java.lang.OutOfMemoryError: GC overhead limit exceeded

    - GC 回收时间过长会抛出 OutOfMemoryError，过长的定义是，超过 98% 的时间用来做 GC并且回收了不到 2% 的堆内存，连续多次 GC都只回收了不到 2%的极端情况下才会抛出，假如不抛出 GC  overhead limit 错误会发生什么呢？那就是 GC 清理的这么点内存很快会再次填满，迫使 GC 再次执行，这样就形成恶性循环，CPU使用率一直是 100%，而 GC 却没有任何成果。

  - java.lang.OutOfMemoryError: Direct  buffer  memory

    - 写 NIO 程序经常使用 ByteBuffer 来读取或者写入内存，这是一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在  Java 堆里面的 DirectByteBuffer 对象作为这块内存的引用进行操作，这样 能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据

    - ByteBuffer.allocate(capability) 第一种方式是分配 JVM 堆内存，属于 GC 管辖范围，由于需要拷贝所以速度较慢
    - ByteBuffer.allocateDirect(capability) 第二种方式是分配OS本地内存，不属于 GC 管辖范围，由于不需要内存拷贝所以速度相对较快
    - 但如果不断分配本地内存，堆内存很少使用，那么 JVM 就不需要执行 GC，DirectByteBuffer 对象们就不会被回收，这时候堆内存充足，但本地内存可能已经使用光了，再次尝试分配本地内存就会出现 OutOfMemoryError，那程序就直接崩溃了

  - java.lang.OutOfMemoryError: unable  to create new native thread

    - 高并发请求服务器时，经常出现这个异常，准确的讲 native thread 异常与对应的平台有关
    - 导致原因：
      - 你的应用创建太多线程了，一个应用进程创建多个线程，超过系统承载极限。
      - 你的服务器并不允许你的应用程序创建这么多线程，Linux系统默认允许单个进程可以创建的线程数量是 1024 个
    - 解决方法：
      - 想办法降低应用程序创建线程的数量，分析应用是否真的需要创建这么多线程，如果不是，改代码将线程数降到最低
      - 对于有的应用，确实需要创建很多线程，远超过 Linux 系统的默认 1024 个线程的限制，可以通过修改 Linux 服务器配置，扩大 Linux 默认限制
      - vim /etc/security/limits.d/90-nproc.conf 发现除了root，其他账户限制在 1024 个

  - java.lang.OutOfMemoryError: Metaspace

    - Metaspace 是方法区在 HotSpot 中的实现，他与持久代最大的区别在于，Metaspace 并不在虚拟机中而是使用本地内存。
    - 永久代存储了以下信息：
      - 虚拟机加载的类信息
      - 常量池
      - 静态变量
      - 即时编译后的代码

- GC 垃圾回收算法和垃圾收集器的关系？分别是什么请你谈谈

  - GC算法：引用计数/复制/标记清除/标记整理，垃圾收集器就是算法落地实现
  - 4 种主要的垃圾收集器：
    - 串行垃圾回收器（Serial）
    - 并行垃圾回收器（Parallel）
    - 并发垃圾回收器（CMS）
    - G1 垃圾回收器

- 怎么查看服务器默认的垃圾收集器是哪个？生产上如何配置垃圾收集器的？谈谈你对垃圾收集器的理解？

  - -XX:+PrintCommandLineFlags
  - -XX:+UseSerialGC / -XX:+UseParallelGC / UseConcMarkSweepGC / UseParNewGC / UseParallelOldGC / UseG1GC
  - Server / Client 模式
    - 32位 Window 操作系统，不论硬件如何都默认使用 Client 的 JVM 模式
    - 32位其它操作系统，2G内存同时有2个cpu以上用 Server 模式，低于该配置还是 Client 模式
    - 64位 only server 模式

- G1 垃圾收集器

  - 见 Java 虚拟机

- 生产环境服务器变慢，诊断思路和性能评估谈谈 / 常用的5个系统命令
  - 整机：top，uptime
  - CPU：vmstat，查看所有cpu核信息：mpstat -P ALL 2，每个进程使用cpu的用量分解信息：pidstat -u 1 -p 进程编号
    - procs	
      - r：运行和等待 CPU 时间片的进程数，原则上1核的 CPU 的运行队列不要超过2，整个系统的运行队列不能超过总核数的2倍，否则代表系统压力过大
      - b：等待资源的进程数，比如正在等待磁盘 I/O，网络 I/O 等
    - cpu
      - us：用户进程消耗 cpu 时间百分比，us值高，用户进程消耗 CPU时间多，如果长期大于 50%，优化程序
      - sy：内核进程消耗 CPU 时间百分比
      - us + sy 参考值为 80%，如果 us + sy 大于80%，说明可能存在 CPU 不足
      - id：处于空闲的 CPU 百分比
      - wa：系统等待 IO的CPU时间百分比
      - st：来自于一个虚拟机偷取的 CPU 时间的百分比
  - 内存：free
  - 硬盘：df
  - 磁盘IO：iostat
  - 网络IO：ifstat



- 加入生产环境出现 CPU 占用过高，请谈谈你的分析思路和定位

  - 结合 Linux 和 JDK 命令一块分析
  - 案例步骤
    - 先用 top 命令找出 CPU 占比最高的
    - ps -ef 或者 jps 进一步定位，得知是一个怎么样的一个后台程序
    - 定位到具体线程或者代码
      - ps  -mp  进程  -o  THREAD,tid,time
      - -m显示所有的线程，-p pid进程使用cpu的时间，-o 该参数后是用户自定义格式
    - 将需要的线程ID转换为16进制格式(英文小写格式)
    - jstack 进程ID | grep tid -A60



### 三. GitHub的骚操作

- 如何找到优秀的源码 + 深度的框架解读 + 学习其它高手的代码 + 自己共享贡献

- 常用词含义

  > watch：会持续收到该项目的动态
  > fork：复制某个项目到自己的 Github 仓库
  > star：可以理解为点赞
  > clone：将项目下载至本地
  > follow：关注你感兴趣的作者，会收到他们的动态

- 关键词索引

  > in：
  >
  > > xxx关键词 in:name/description/readme，关键词在项目名/描述、readme文件中。组合使用则用逗号隔开 seckill in:name,readme
  >
  > stars 和 forks
  >
  > > xxx关键字 stars :> / :>= / :num1..num2，sprintboot stars:>=5000
  >
  > awesome
  >
  > > awesome 关键字：一般用来收集学习，工具，书籍类相关的项目
  > > awesome redis：搜索优秀的 redis 相关的项目，包括框架，教程等
  >
  > 高亮显示
  >
  > > 地址#Lnum1-num2
  >
  > 项目内搜索
  >
  > > 英文 t
  > > https://help.github.com/en/articles/using-keyboard-shortcuts
  >
  > 搜索某个地区的大佬
  >
  > > location:地区  language:语言
  > > 北京地区的Java方向的用户: location:beijing laguage:java



### 四. Spring = IOC + AOP + TX

#### 1. spring的aop顺序

- Aop常用注解
  - @Before，前置通知，目标方法之前执行
  - @After，后置通知，目标方法之后执行（始终执行）
  - @AfterReturning，返回后通知，执行方法结束前执行（异常不执行）
  - @AfterThrowing，异常通知，出现异常时执行
  - @Around，环绕通知，环绕目标方法执行
- Spring4 + springboot1.5.9，aop正常顺序 + 异常顺序
  - 正常执行：环绕前 ==> @Before ==> 环绕后 ==> @After ==> @AfterReturning
  - 异常执行：环绕前 ==> @Before ==> @After ==> @AfterThrowing
- Spring5 + springboot2.3.3
  - 正常执行：环绕前 ==> @Before ==> @AfterReturning ==> @After ==> 环绕后
  - 异常执行：环绕前 ==> @Before ==> @AfterThrowing ==> @After

#### 2. spring循环依赖

- 循环依赖是什么

  > 多个 bean 之间相互依赖，形成了一个闭环，比如: A依赖于B，B依赖于C，C依赖于A。
  > 我们AB循环依赖问题只要A的注入方式是 setter 且 singleton 就不会有循环依赖问题
  > 默认的单例(singleton)的场景是支持循环依赖的，不报错
  > 原型(Prototype)的场景是不支持循环依赖的，会报错
  
- spring内部通过3级缓存来解决循环依赖

  - DefaultSingletonBeanRegistry
    - Map<String, Object>  singletonObjects = new ConcurrentHashMap()，一级缓存，存放已经经历了完整生命周期的 Bean 对象
    - Map<String, Object>  earlySingletonObjects = new HashMap()，二级缓存，存放早期暴露出来的 Bean 对象，Bean的生命周期未结束（属性还未填充）
    - Map<String, Object<Factory<?>>  singletonFactories = new HashMap()，三级缓存，存放可以生成 Bean 的工厂

  - <span style="color:red">只有单例的 bean 会通过三级缓存提前暴露来解决循环依赖的问题，而非单例的 bean，每次从容器中获取都是一个新的对象，都会重新创建，所以非单例的 bean 是没有缓存的，不会将其放到三级缓存中</span>。

  

#### 3. 大厂面试题

- spring中的三级缓存



- 三级缓存分别是什么？三个Map有什么异同



- 什么是循环依赖？看过spring源码吗？



- 如何检测是否存在循环依赖？实际开发中见过循环依赖吗？



- 多例的情况下，循环依赖问题为什么无法解决？