## JAVASE

1. 



## JVM（看博客）

1. jvm内存模型：五个部分详细介绍

   1. 虚拟机栈抛异常：

      1. **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
      2. **`OutOfMemoryError`：** Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

   2. 是不是对象一定在堆上分配？

      错误。有可能通过逃逸分析在栈上直接分配；可能在线程内存中的tlab块中分配（避免多线程冲突）。

   3. 为什么需要将永久代替换成元空间？

      jvm管理永久代也是像管理java堆一样，是有一个jvm管理内存的上限的，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

2. 对象的创建过程：

   1. 加载相应的类。
   2. 对对象分配内存
      - 指针碰撞/内存记录
      - cas在堆中分配
      - tlab分配
      - 栈分配
   3. 初始化对象中的值
   4. 初始化对象头
   5. `invokespecial`，真正执行java构造方法。

3. String类和字符串常量池（8之前和8之后）

4. 八种基本类型（自动装箱和缓存机制）

5. 堆的内存分配：<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/jvm%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6/01d330d8-2710-4fad-a91c-7bbbfaaefc0e.png" alt="img" style="zoom: 50%;" />（新生代和老年代默认比例1:2，eden:from:to  8:1:1）

6. 触发gc的时机：

   - 当eden区满了的时候，需要进行`young gc`。
   - 当old区满了的时候，需要触发full gc。

7. 如何判断对象需要回收？

   1. 引用计数法：无法解决循环引用问题

   2. 可达性分析：jvm使用的方法，先枚举根节点，通过根节点进行可达性判断。

      > 哪些可以作为根节点？
      >
      > - 虚拟机栈(栈帧中的本地变量表)中引用的对象
      > - 本地方法栈(Native 方法)中引用的对象
      > - 方法区中类静态属性引用的对象
      > - 方法区中常量引用的对象
      > - 所有被同步锁持有的对象

8. 如何判断方法区中的类型信息需要回收？（jvm可以不对其实现垃圾回收）

   1. 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
   2. 加载该类的 `ClassLoader` 已经被回收。
   3. 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

9. 三种垃圾收集算法，分别用在什么区域。

   > 与标记-清除的侧重点不一样，在标记-清除主要关注的是垃圾收集的STW延迟，而标记-整理主要关注的是吞吐量。

   详见jvm垃圾收集...

10. 四大引用以及相应原理。

11. 

12. 如何判断对象已经死亡？

    1. 引用计数法：不能解决循环引用问题
    2. 可达性分析法：通过对根节点枚举，进行对象的可达性分析。

    

    

    

    

    







## 并发编程/JUC

### 基础

1. 线程的生命周期和状态：

   1. new：创建线程

   2. Runable：线程正在运行中或者Ready（等待操作系统调度）

      > 操作系统隐藏 Java 虚拟机（JVM）中的 READY 和 RUNNING 状态，它只能看到 RUNNABLE 状态

   3. blocked：阻塞，可能是io阻塞或者锁阻塞

   4. waiting：等待其他线程通知（notify）

   5. TIME_WAITING：超时等待，当时间到了可以重新进入

   6. terminal：终止，线程结束

2. 几种创建线程的方式：

   - 继承thread，重写Thread类的run方法
   - 送入runnable
   - 使用FutureTask配合callable来实现有返回值的线程：`new Thread(new FutureTask(new Callable {...}))`

3. 多线程可能造成的问题：内存泄漏、死锁、线程不安全等等。

4. sleep和wait：

   1. sleep没有释放锁、wait释放锁
   2. sleep只是对当前线程的暂停执行，wait用于线程间的交互通信（通过notify唤醒）

5. 线程安全的定义：多个线程同时访问时，其表现出正确的行为。通俗一点说，就是假设这些线程都是串行执行的，并行情况下执行出来的值如果都是属于串行执行的值的子集，认为就是线程安全的。

6. 并发和并行：

   - 并发：同一个时间段多个线程同时执行（一个cpu核执行多个任务）
   - 并行：同一时间多个线程同时执行（多核同时执行不同的线程）

7. 线程间通信：

   1. wait/notify/notifyAll

   2. 使用Condition  

      > 详见另一篇juc

### JUC

1. 并发编程的重要特性

   1. **原子性** : 一个的操作或者多次操作，要么所有的操作全部都得到执行并且不会收到任何因素的干扰而中断，要么所有的操作都执行，要么都不执行。`synchronized` 可以保证代码片段的原子性。
   2. **可见性** ：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。`volatile` 关键字可以保证共享变量的可见性。
   3. **有序性** ：代码在执行的过程中的先后顺序，Java 在编译器以及运行期间的优化，代码的执行顺序未必就是编写代码时候的顺序。`volatile` 关键字可以禁止指令进行重排序优化。

2. volatile

   1. 作用：轻量级的锁，能够保证不同线程对共享变量的可见性、指令重排
   2. 具体实现：
      1. jmm内存模型：工作内存、主内存；映射到cpu也就是cpu缓存、主内存；
      2. 底层通过lock指令
         1. 写的时候锁系统总线，使得同一时间只有自己能够更新，保证轻量级的原子性；
         2. 主动将cpu缓存同步到主内存中，别的线程会触发缓存一致性协议，更新此时的缓存，实现可见性；
         3. lock指令防止指令重排，产生cpu级别的内存屏障；

3. synchronized：

   - 作用：保证不同线程的同步执行（排他锁）、并且保证一定程度的可见性和指令重排。（具体实现说一下）

   - 字节码层面：

     **`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

     在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

     在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

   - 早期版本：重量级锁，原子性保证：使用了操作系统的mutex互斥量

     > 互斥量实现：当这个空间被赋值为1的时候表示加锁了，被赋值为0的时候表示解锁了，仅此而已。多个线程抢一个锁，就是抢着要把这块内存赋值为1。在一个多核环境里，内存空间是共享的。每个核上各跑一个线程，那如何保证一次只有一个线程成功抢到锁呢？你或许已经猜到了，这必须要硬件的某种guarantee。
     >
     > CPU如果提供一些用来构建锁的atomic指令，譬如x86的CMPXCHG（加上LOCK prefix），能够完成atomic的[compare-and-swap](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Compare-and-swap) （CAS），用这样的硬件指令就能实现[spin lock](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Spinlock)。本质上LOCK前缀的作用是锁定系统总线（或者锁定某一块cache line）来实现atomicity，可以了解下基础的缓存一致协议譬如MSEI。简单来说就是，如果指令前加了LOCK前缀，就是告诉其他核，一旦我开始执行这个指令了，在我结束这个指令之前，谁也不许动。缓存一致协议在这里面扮演了重要角色，这里先不赘述。这样便实现了一次只能有一个核对同一个内存地址赋值。

   - synchronized实现可见性和指令重排序：

     - 实现可见性

       synchronized在获取了锁之后，首先会对工作内存中的共享变量进行同步操作(lock)，在出同步代码块的时候，会将工作内存重新同步到主内存，也就是说synchronized在执行同步代码块中的时候，其实是不能保证共享变量和其他线程的可见性的，他能保证的也就是不同线程在同步代码块中的可见性。

     - 实现指令重排序

       上面说了，进入的时候，出去的时候，其实都会加汇编级别的lock操作，实现了几段内存屏障。

   - 单例模式：

     ```java
     public class Singleton {
     
         private volatile static Singleton uniqueInstance;
     
         private Singleton() {
         }
     
         public  static Singleton getUniqueInstance() {
            //先判断对象是否已经实例过，没有实例化过才进入加锁代码
             if (uniqueInstance == null) {
                 //类对象加锁
                 synchronized (Singleton.class) {
                     if (uniqueInstance == null) {
                         uniqueInstance = new Singleton();
                     }
                 }
             }
             return uniqueInstance;
         }
     }
     ```

     > 为什么需要加volatile：因为说白了synchroinzed虽然能防止指令重排序，但是他只是在进入同步代码块和出同步代码块的时候加入了内存屏障，在同步代码块里面有可能会进行重排序，而new对象不是一个原子操作，所以有可能先赋值地址再进行初始化，如果赋值地址完了cpu此时进行了切换，那么其他线程get的时候uniqueInstance != null，但是其实此时并没有完成初始化，所以会造成空指针异常。

   - synchronized1.6版本以上的优化：

     - 引入了自旋锁、偏向锁、轻量级锁，主要是根据不同并发强度来提供相应锁的支持。
     - 几种锁适用的并发场景：
       - 偏向锁：一个线程出同步代码块了，另一个线程才进来；保证无竞争下的同步，提高性能。
       - 轻量级锁：不同线程交替进入，一个线程还没出去，另一个线程才进来。他使用cas操作来保证在没有同时的多线程竞争抢锁的前提下，避免进入操作系统mutex的性能消耗。
       - 重量级锁：适用于多个线程抢锁，同时想进入临界区。
     - 底层原理：通过cas，不同线程操作markword，通过cas操作的成功与否来实现此时的并发强度的判断和锁的升级。
       - 无锁情况进入同步代码块：判断`markword`此时无锁状态，cas将当前`markword`设置为偏向锁，偏向id就是当前线程id
       - 偏向锁进入同步代码块：判断`markword`此时的线程id和本线程id是否相同，相同就不管；如果不相同，此时会让markword中的thread线程走到安全点的时候进行判断此时此线程出了同步代码块没有，如果出了的话就重置markword，使得markword偏向当前线程，如果没有出此时就升级为轻量级锁。
       - 轻量级锁进入同步代码块：此时会一直进行cas自旋操作，如果此时持有锁的线程出了同步代码块，会通过cas将当前栈中的markword进行替换，此时请求锁的线程就会cas成功；如果自旋太久，那么会升级为重量级锁。

   - `synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

     - **`volatile` 关键字**是线程同步的**轻量级实现**，所以 **`volatile `性能肯定比`synchronized`关键字要好** 。但是 **`volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块** 。
     - **`volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。**
     - **`volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。**

   - `synchronized`和`ReenLock`区别：

     - synchronized是一个java内置的语法，Lock是一个类
     - synchronized是自动的，Lock要手动释放锁
     - synchronized是可重入锁，并且不可中断，非公平；Lock是可重入锁，可以判断锁的状态，非公平（能够自己设置）；
     - synchronized只要被一个线程获得了锁，就算阻塞了，别的线程也会傻傻等下去；Lock不会，它有判断机制。
     - lock在锁等待的时候可以被中断（使用`lock.lockInterruptibly()`或者`tryLock()`的时候）

4. ThreadLocal

   - 作用：让每个线程绑定自己的私有值。

   - 实现：每个线程有一个`ThreadLocalMap`，`ThreadLocalMap`中以当前的`ThreadLocal`为键，存储相应数据。

   - 问题：内存泄漏问题

     > `ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法

5. cas：比较并交换，是cpu级别的原子操作。

   - 实现：有三个值，包括当前内存的值、期待的值、要更新的值，CAS操作：如果当前内存中的值和期待的值相等，就set为要更新的值。

   - java实现：通过`Unsafe`来实现的：

     ```java
         private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
     		// 拿到value对应的位置，也就是主内存中的位置。
         private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
     
         private volatile int value;
     ```

   - 应用：

     - 原子类：通过CAS+自旋

       每次自增的时候，先从偏移处拿出此时的主内存值到工作内存，再使用cas判断此时工作内存值和主内存值是否相等，如果相等就更新，如果不相等就一直自旋操作；

   - ABA问题：比较并交换的时候，有可能别的线程将a变成b再变成a，此时cas还是能成功。

     ​	解决：通过使用版本号解决，使用`AtomicStampedReference`

6. AQS：AbstractQueuedSynchronizer

   - 定义：AQS 是一个用来构建锁和同步器的框架，底层也是基于cas，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

   - 设计思想：理解为是基于对state共享变量的一系列操作，从而实现线程的唤醒和阻塞的一套机制；基于CLH队列来实现的。

   - 资源共享方式：

     - 独占式：`ReentrantLock`
     - 共享式：`CountDownLatch`等

   - 底层实现：模板方法模式

     - isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。 
     - tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。 
     - tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。 
     - tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。 
     - tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。

     >  实现同步类的时候，只需要实现`tryRelease-tryAcquire`和`tryAcquireShared-tryReleaseShared`中的一组即可。

   - 底层实现大致流程：当获取资源失败的时候，会添加入相应的等待队列中（通过`LockSupport.park(this);`实现）；当释放资源成功的时候，会将队列中的元素进行检索`doReleaseShared();`，将当前队列头线程进行解锁`LockSupport.unpark(s.thread);`，将之前线程唤醒，此时线程肯定是在队列的头节点中，直接申请资源、移除队列，唤醒执行即可。

   - 应用：

     - `CountDownLatch`

     - `CyclicBarrier`

       > 对于 `CountDownLatch` 来说，重点是“一个线程（多个线程）等待”，而其他的 N 个线程在完成“某件事情”之后，可以终止，也可以等待。而对于 `CyclicBarrier`，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。CountDownLatch` 是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而 `CyclicBarrier` 更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

     - **`Semaphore`**

     - `SynchronousQueue`

7. 线程池

   - `Runnable`和`Callable`区别：

     - callable可以抛出异常、并且可以有返回值。

       > 原理：
       >
       > 其实就是在Runable上封装了一层FutureTask，当执行FutureTask的run方法的时候会调用Callable的call()，并且在执行FutureTask的run的过程中，拿到call的返回值并set到本对象的成员变量中，此时get就解除阻塞，拿到相应的值。
       >
       > 如果子线程发生了异常，那么FutureTask#run也会进行相应的捕捉，全局设置异常，get出来的时候就能判断返回，在主线程中进行throw。

   - 创建方式：

     - `new ThreadPoolExecutor`

       ```java
       public ThreadPoolExecutor(int corePoolSize,  // 核心线程数，也就是初始线程数
                             int maximumPoolSize,   // 最大线程数，也就是当队列满了以后，触发最大线程数
                             long keepAliveTime,  // 当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；
                             TimeUnit unit,  // 时间单位
                             BlockingQueue<Runnable> workQueue,  // 任务队列
                             ThreadFactory threadFactory,  // 创建线程方式
                             RejectedExecutionHandler handler  // 拒绝策略，当此时是最大线程了，并且任务队列满了触发
                            )
       ```

       

     - 通过`Executers`创建（不能配置参数，不推荐）：

       - **FixedThreadPool** ： 该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
       - **SingleThreadExecutor：** 方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
       - **CachedThreadPool：** 该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。

   - 提交方式：

     - **`execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
     - **`submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

   - 拒绝策略：

     - **`ThreadPoolExecutor.AbortPolicy`：** 抛出 `RejectedExecutionException`来拒绝新任务的处理。
     - **`ThreadPoolExecutor.CallerRunsPolicy`：** 调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
     - **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
     - **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。

   - 并发容器

   - java8并发流



















