1.volatile 
------
    在多处理器下，为了保证各个处理器的缓存是一致的，就会实现**缓存一致性**协议，**每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期**了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。
    
    1.1 在生成汇编代码时会对volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令，Lock前缀的指令会引起处理器缓存写回内存。
    1.2 一个处理器的缓存回写到内存会导致其他处理器的缓存失效。
    1.3 当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。
      volatile变量通过这样的机制就使得每个线程都能获得该变量的最新值。
      
    volatile：jdk提供的一种轻量级的同步机制，保证不同线程间对共享变量操作的可见性，阻止编译器和处理器的指令重排(通过添加JMM内存屏障)，但是不保证原子性
      
 2.CAS
 ------
    compare and swap保证变量的原子性操作
    CAS机制中，使用三个操作数：内存地址V，旧的预期值A，要修改的新值B
    更新一个变量的时候，只有当旧的预期值A和内存地址V中的实际值相同时，才会将内存地址V对应的值修改为B
    底层实现：利用unsafe提供的原子性操作方法，unsafe是底层硬件级的操作
    
    CAS通过自旋实现，1.获取当前值2.当前值+1，计算出目标值3.进行CAS操作，成功就跳出循环，失败就重复上述步骤
    缺点：CPU开销过大，不能保证代码块的原子性，ABA问题
    ABA问题：线程1：A->B
             线程2：阻塞
             线程3：B->A
             线程2：阻塞完成，A->B
             
    ABA问题解决：值+版本号
                例如：AtomicStampedReference类实现了带版本号的CAS机制

3.公平锁+非公平锁
------
    公平锁：指多个线程按照申请锁的顺序来获取锁，先来后到，早到早得
    非公平锁：指多个线程获取锁的顺序并非按照申请锁的顺序，有可能后申请的线程比先申请的线程要更早的获取到锁。
    非公平锁的优点在于吞吐量比公平锁大
    Synchronized锁是一种非公平锁
    ReentrantLock锁可以根据析构函数的参数决定是公平锁还是非公平锁。
  
4.可重入锁  
------
    作用是防止死锁  
    线程获取锁后可以重复执行锁区域。Java提供的锁都是可重入锁。不可重入锁非常容易导致死锁。  
5.共享锁
------
    线程可以同时获取锁。ReentrantReadWriteLock对于读锁是共享的，在读多写少的情况下使用共享锁会非常高效。  
6.排它锁
------
    多线程不可同时获取的锁。与共享锁对立。与可重入锁不矛盾可以是并存属性。  
7.偏向锁  
------
    一段同步代码一直被一个线程所访问，那么该线程会自动获取所。降低获取锁的代价，类似于乐观锁。  
8.轻量级锁  
------
    当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的方式尝试获取锁，不会阻塞，提高性能。  
9.重量级锁  
------
    当锁是轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁会膨胀为重量级锁，重量级锁会让其他申请的线程进入阻>>塞， 性能降低。  
10.分段锁  
------
    分段锁是一种锁思想，对数据分段加锁提高并发效率，比如jdk8之前的ConcurrentHashMap(采用分段锁+数组+链表)，jdk8之后CAS+Synchronized。  
    当需要PUT元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道它要放在哪一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，  
    就实现了真正的并行插入。  
    分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅对数组中的一项进行加锁操作。  
11.自旋锁  
------
    是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU  
12.CountDownLatch
------
    CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说多个线程之间的通信。
    CountDownLatch能够使一个线程在等待另外一些线程完成各自的工作后，再继续执行。使用一个计数器实现，计数器初始值为线程的数量，当一个线程完成自己的任务后，计数器的值就会减1，当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务。
    CountDownLatch的典型用法：某一线程开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为new CountDownLatch(n),每当一个线程执行完毕，就将计数器减1，countDownLatch.countDown(),当计数器的值变成0时，在CountDownLatch上await()的线程就会被唤醒。
    CountDownLatch的应用场景就是：启动一个服务的时候，主线程需要等待多个组件加载完毕，再继续执行。
    缺点：
        CountDownLatch是一次性的，初始化的计数器的值，只能初始化一次，不能重复利用。
 13.阻塞队列
 ------
    BlockingQueue
    阻塞队列区别于普通队列的就在于阻塞：当线程想要消费队列中的数据时，如果队列为空，那么线程将会被阻塞挂起，直到队列中出现了数据。
    阻塞队列分为先进先出和后进先出两种策略。
    BlockingQueue的核心方法：
    1.放入数据
        （1）offer(anObject):表示如果可能的话,将anObject加到BlockingQueue里,即如果BlockingQueue可以容纳,则返回true,否则返回false.（本方法不阻塞当前执行方法的线程）；　　　　　　 
     　 （2）offer(E o, long timeout, TimeUnit unit)：可以设定等待的时间，如果在指定的时间内，还不能往队列中加入BlockingQueue，则返回失败。                     
         （3）put(anObject):把anObject加到BlockingQueue里,如果BlockQueue没有空间,则调用此方法的线程被阻断直到BlockingQueue里面有空间再继续。
     2. 获取数据
        （1）poll(time):取走BlockingQueue里排在首位的对象,若不能立即取出,则可以等time参数规定的时间,取不到时返回null;
        （2）poll(long timeout, TimeUnit unit)：从BlockingQueue取出一个队首的对象，如果在指定时间内，队列一旦有数据可取，则立即返回队列中的数据。否则知道时间超时还没有数据可取，返回失败。           
        （3）take():取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到BlockingQueue有新的数据被加入; 
        （4）drainTo():一次性从BlockingQueue获取所有可用的数据对象（还可以指定获取数据的个数），通过该方法，可以提升获取数据效率；不需要多次分批加锁或释放锁。
                3. BlockingQueue的实现
                3.1 ArrayBlockingQueue基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。
                    ArrayBlockingQueue在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue；按照实现原理来分析，ArrayBlockingQueue完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea之所以没这样去做，也许是因为ArrayBlockingQueue的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。 ArrayBlockingQueue和LinkedBlockingQueue间还有一个明显的不同之处在于，前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。而在创建ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁
                3.2 LinkedBlockingQueue基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而LinkedBlockingQueue之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
                        作为开发者，我们需要注意的是，如果构造一个LinkedBlockingQueue对象，而没有指定其容量大小，LinkedBlockingQueue会默认一个类似无限大小的容量（Integer.MAX_VALUE），这样的话，如果生产者的速度一旦大于消费者的速度，也许还没有等到队列满阻塞产生，系统内存就有可能已被消耗殆尽了。
                3.3 DelayQueue：DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。
                    DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞
                3.4 PriorityBlockingQueue：基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁
                3.5 SynchronousQueue 一种无缓冲的等待队列，类似于无中介的直接交易，有点像原始社会中的生产者和消费者，生产者拿着产品去集市销售给产品的最终消费者，而消费者必须亲自去集市找到所要商品的直接生产者，如果一方没有找到合适的目标，那么对不起，大家都在集市等待。相对于有缓冲的BlockingQueue来说，少了一个中间经销商的环节（缓冲区），如果有经销商，生产者直接把产品批发给经销商，而无需在意经销商最终会将这些产品卖给那些消费者，由于经销商可以库存一部分商品，因此相对于直接交易模式，总体来说采用中间经销商的模式会吞吐量高一些（可以批量买卖）；但另一方面，又因为经销商的引入，使得产品从生产者到消费者中间增加了额外的交易环节，单个产品的及时响应性能可能会降低。
                声明一个SynchronousQueue有两种不同的方式，它们之间有着不太一样的行为。公平模式和非公平模式的区别:
                如果采用公平模式：SynchronousQueue会采用公平锁，并配合一个FIFO队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；
                但如果是非公平模式（SynchronousQueue默认）：SynchronousQueue采用非公平锁，同时配合一个LIFO队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。

14.synchronized和lock的区别
------
        1.synchronized是java的关键字，在jvm层面上；lock是一个接口
        2.synchronized无法判断是否获取锁的状态，Lock可以判断是否获取到锁
        3.synchronized可以自动释放锁（a线程执行完同步代码会释放锁，b线程执行过程中发生异常会释放锁），Lock需要在finally中手动释放锁(使用unLock()方法释放锁)，否则会造成死锁
        4.使用synchronized的两个线程，线程1和线程2，当线程1获取到锁，线程2等待，如果线程1阻塞，线程2会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程不会一直等待就结束了。
        5.synchronized锁不可重入，不可中断，非公平，而Lock锁可重入，可中断，可公平
        6.Lock适合大量同步的代码的同步问题，synchronized锁适合少量的代码的同步问题
        7.synchronized使用Object对象本身的wait、notify、notifyAll调度机制，而Lock可以使用Condition进行线程间的调度(Condition定义了等待/通知两种类型的方法)
                Lock lock = new ReentrantLock();
                Condition condition = lock.newCondition();
                ...
                condition.await();
                ...
                condition.singnal();
                condition.signalAll();
        
        Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：
         1.普通同步方法，锁是当前实例对象
         2.静态同步方法，锁是当前类的class对象
         3.同步方法块，锁是括号里面的对象
        同步代码块：monitorenter指令是在编译之后，插入同步代码块的开始位置，monitorexit指令插入同步代码块的结束位置，JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor所有权，即尝试获取对象的锁；
        同步方法：synchronized方法则会被翻译成普通的方法调用和返回指令如:invokevirtual、areturn指令，在VM字节码层面并没有任何特别的指令来实现被synchronized修饰的方法，而是在Class文件的方法表中将该方法的access_flags字段中的synchronized标志位置1，表示该方法是同步方法并使用调用该方法的对象或该方法所属的Class在JVM的内部对象表示Klass做为锁对象。
        
15.Callable接口
------
        Callable接口：返回结果并且可能抛出异常的任务。
        优点：
            1.可以获得任务执行返回值
            2.通过与Future的结合，可以实现利用Future来跟踪异步计算的结果
            
         Runnable接口和Callable接口的区别：
            1.Callable规定的方法是call(),Runnable规定的方法是run()
            2.Callable的任务执行后可返回值，Runnable的任务不能返回值
            3.call()方法可以抛出异常，run()方法不能
            4.运行Callable任务可以拿到一个Future对象，表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并检查计算结果。通过Future对象可以了解任务执行的情况，可取消任务的执行，还可获取执行结果。
            5.代码实例：
                    public interface Callable<V>{
                        V vall() throws Exception;
                    }
                    public interface Runnable{
                        public abstract void run();
                    }
            6.Future接口
                Future是一个接口，代表了一个异步计算的结果。接口中的方法用来检查计算是否完成、等待完成和得到计算的结果。
                当计算完成后，只能通过get()方法得到结果，get方法会阻塞直到结果准备好了。
                如果想取消，那么调用cancel()方法。其他方法用于确定任务是正常完成还是取消了。
                一旦计算完成了，那么这个计算就不能被取消。
16.线程池使用及优势
------
            1.线程池的概念
                线程池做的主要工作就是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果运行线程数量超出了最大线程数量，超出部分需要排队等候，等待其他线                     程执行完毕，然后线程再从队列中取出任务执行。
            2.线程池的主要特点
                2.1 线程复用
                2.2 控制最大并发数
                2.3 管理线程
            3.线程池的优势
                3.1 降低资源消耗，通过重复利用已经创建的线程降低创建线程以及销毁线程的消耗
                3.2 提高响应速度，当任务到达时，任务可以不用等待线程创建就能立即执行
                3.3 提高线程的可管理性，线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
            4.线程池7大参数理解
                4.1 核心线程(corePoolSize)
                    线程池最终执行任务的角色肯定是线程，同时我们也会限制线程的数量，所以我们可以这样理解核心线程，有新任务提交时，首先检查核心线程数，如果核心线程都在工作，而且数量                        已经达到最大核心线程数，则不会继续新建核心线程，而会将任务放入等待队列。
                4.2 等待队列(workQueue(jdk提供四种工作队列：ArrayBlockingQueue、LinkedBlockingQuene、SynchronousQuene、PriorityBlockingQueue))
                    等待队列用于存储当核心线程都处于运行状态时，继续新增的任务。核心线程在执行完当前任务后，会去等待队列拉取任务继续执行，这个队列一般是一个线程安全的阻塞队列，它的                        容量是由开发者根据业务来定制。
                4.3 非核心线程(maximumPoolSize)：
                    当等待队列满了，如果当前线程数没有超过最大线程数，则会新建线程执行任务，其实本质上核心线程和非核心线程没有区别。创建出来的线程也没有标识去区分它们是核心的还是非                        核心的，线程池只会去判断已有的线程数（包括核心和非核心）去跟核心线程数和最大线程数比较，来决定下一步的策略。
                4.4 线程活动保持时间（keepAliveTime）：线程空闲下来之后，保持存活的持续时间，超过这个时间还没有任务执行，该工作线程结束。
                4.5 unit 空闲线程存活时间单位
                    keepAliveTime的计量单位
                4.6 threadFactory 线程工厂
                    创建一个新线程时使用的工厂，可以用来设定线程名、是否为daemon线程等等
                4.7 handler 拒绝策略（RejectedExecutionHandler）：当等待队列已满，线程数也已达最大线程数，线程池会根据饱和策略来执行后续的操作，默认策略是抛弃要加入的任务。
                    jdk提供了4种拒绝策略
                        ①CallerRunsPolicy：该策略下，在调用者线程中直接执行被拒绝任务的run方法，除非线程池已经shutdown，则直接抛弃任务
                        ②AbortPolicy：该策略下，直接丢弃任务，并抛出RejectedExecutionException异常。
                        ③DiscardPolicy：该策略下，直接丢弃任务，什么都不做。
                        ④DiscardOldestPolicy：该策略下，抛弃进入队列最早的那个任务，然后尝试把这次拒绝的任务放入队列
                        
线程池运作流程图：![线程池运作流程图](https://github.com/yushen0/review/blob/main/images/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E8%BF%90%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)
                
                当有新任务提交到线程池时，线程池的处理流程如下：
                ①判断核心线程池是否已满。如果没满，则创建新线程来执行此任务（即使当前有空闲的线程也会直接创建，而不是使用空闲线程来执行），直到核心线程池中的线程数达到了设置的大小之                     后就不再创建；如果核心线程池已满，则进入下一阶段的判断。
                ②判断等待队列是否已满。如果没满，则将任务暂时存放到等待队列中，等待核心线程池中的线程空闲下来再来获取任务执行（核心线程池中的线程执行完任务之后会循环从等待队列中取任                     务来执行）；如果队列已满，则进入下一阶段的判断。
                ③判断线程池是否已满。（线程池除了核心线程池，还设置了线程池的最大线程大小，即使核心线程池满了，还可以再创建线程），如果线程池中工作的线程没有达到最大值，则创建新线程                     来执行任务；如果线程池已满，则按照饱和策略来处理任务。
                使用注意事项：
                ①线程池的大小：并非越多越好。应根据系统运行的硬件环境以及应用本身的特点来决定线程池的大小。一般来说，如果代码结构合理，线程数与cpu数量相适合即可。如果线程运行时可能                    出现阻塞现象，可响应增加线程池的大小，如果有必要可以采用自适应算法来动态调整线程池的大小，以提高cpu的有效利用率和系统的整体性能。
                ②并发错误：多线程应用要特别注意并发错误，要从逻辑上保证程序的正确性，注意避免死锁现象的发生。
                ③线程泄露：这是线程池应用中的一个严重的问题，当任务执行完毕而线程没能返回线程池中就会发生线程泄露现象。
                
              5 线程池的三种常用方式：
                    5.1 Executors.newFixedThreadPool(int) 使用场景： 执行长期的任务，性能好很多
                            public static ExecutorService newFixedThreadPool(int nThreads) {
                                    return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
                             }
                         主要特点如下：
                            创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
                            使用newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的是LinkedBlockingQueue；
                    5.2 Executors.newSingleThreadExecutor() 使用场景：一个任务一个任务执行的场景
                            public static ExecutorService newSingleThreadExecutor() {
                                     return new FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>()));
                             }
                          主要特点如下：
                            创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按顺序执行
                            使用newSignleThreadExecutor将corePoolSize和maxminumPoolSize都设置为1，它使用的是LinkedBlockingQueue;
                    5.3 Executors.newCachedThreadPool()   使用场景：短期异步的小程序，或者负载较轻的服务器
                            public static ExecutorService newCachedThreadPool() {
                                    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>());
                             }
                          主要特点：
                            创建一个可缓存的线程池，如果线程池长度超过处理需要 ，可灵活回收空闲线程，若无可回收，则新建线程。
                            使用newCachedThreadPool 将 corePoolSize 设置为 0 ，将maximumPoolSize设置为Integer.MAX_VALUE, 使用的SynchronousQueue也就是说来了任务就创建线程运行，当                                  线程空闲超过60s，就自动销毁线程。
                    5.4 Executors.newScheduledThreadPool() 创建固定大小的线程，可以延迟或定时的执行任务。
                    5.5 java8新出的Executors.newWorkStealingPool(int) 使用目前机器上可用的处理器作为它的并行级别


17.
        
