本章介绍 Java 并发包中相关的 API 和组件，以及这些 API 和组件的使用方式和实现细节。

![](http://img.070077.xyz/202204270249199.png)


# 锁

## Lock接口

Lock 接口出现之前，Java 程序是靠 synchronized 关键字实现锁功能的，而 Java SE 5 之后，并发包中新增了 Lock 接口（以及相关实现类）用来显式实现锁功能。

```java
Lock lock = new ReentrantLock();
lock.lock();
try {...} finally {lock.unlock();}
```

![](http://img.070077.xyz/202203290034087.png)

## 队列同步器AQS

`AbstractQueuedSynchronizer` （简称**同步**器），是用来构建锁或者其他同步组件的基础框架，它使用了一个 int 成员变量表示同步状态，通过内置的 FIFO 队列来完成资源获取线程的排队工作。子类通过**继承**同步器并实现它的抽象方法来管理同步状态，使用同步器提供的 3 个方法 getState 、setState(int newState) 和 compareAndSetState(int expect, int update) 控制状态。

### 实现分析

- 同步队列
  ![](http://img.070077.xyz/202204270257429.png)

同步器会将*当前线程以及等待状态等信息*构造成为一个节点将其加入同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。首节点的线程在释放同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。

![](http://img.070077.xyz/202204270258845.png)


- 独占式同步状态获取与释放
调用同步器的 `acquire(int arg)` 方法可以获取同步状态，该方法对中断不敏感，线程获取同步状态失败后进入同步队列中者，后续对线程进行中断操作时，线程**不会**从同步队列中移出。
![](http://img.070077.xyz/202204270300007.png)

![](http://img.070077.xyz/202204270300980.png)


在获取同步状态时，同步器维护一个同步队列，获取状态失败的线程都会被加入到队列中并在队列中进行自旋；移出队列（或停止自旋）的条件是**前驱节点为头节点且成功获取了同步状态 。** 在释放同步状态时，同步器调用 `tryRelease(int arg)` 方法释放同步状态，然后唤醒头节点的后继节点 .

- 共享式同步状态获取与释放
  共享式获取与独占式获取最主要的区别在*同一时刻能否有多个线程同时获取到同步状态* 。 
  类似独占式，通过`tryAcquireShared(int arg)` 获取，也需要释放同步状态。

- 独占式超时获取同步状态
  同步器提供了 `acquirelnterruptibly(int arg)` 方法，这个方法在等待获取同步状态时，如果当前线程被中断，会立刻返回，并抛出 InterruptedException 。
  ![](http://img.070077.xyz/202204270304804.png)


## 重入锁ReentrantLock

解决这个问题：获取锁之后，如果再次调用 `lock`方法，则该线程将会被自己所阻塞。
- `synchronized` 关键字隐式支持重入
- `ReentrantLock` 通过组合自定义同步器来实现锁的获取与释放。在调用`lock`方法时，已获取到锁的线程，能够再次调用 `lock`方法获取锁而不被阻塞

>`Synchronized` 是JVM层面的锁，是非公平锁。 `Synchronized` 在线程进入 ContentionList 时，等待的线程会先尝试自旋获取锁，如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占 OnDeck 线程的锁资源，不可以被中断。非公平性锁刚释放锁的线程再次获取同步状态的几率会非常大，可能使线程饥饿，但因极少的线程切换，保证了其更大的吞吐量成为默认。

ReentrantLock 在Monitor中有一个计数器，记录重入次数。并提供了一个构造函数，能够控制锁是否是公平的(锁获取是**时间顺序**的，判断同步队列中当前节点是否有前驱节点)。

## 读写锁
读写锁维护了 *一对*锁，分离读锁和写锁。在读多于写的情况下，读写锁能够提供比上述排它锁更好的并发性和吞吐量。

![](http://img.070077.xyz/202204271013808.png)

读写锁的自定义同步器需要在同步状态（int）上维护多个读线程和一个写线程的状态，采用了*高 16 位表示读，低 16 位表示写* 的分离计数方法。

如果存在读锁，则写锁不能被获取，**确保写锁的操作对读锁可见**。写锁一旦被获取，则其他读写线程的后续访问均被阻塞 。

支持锁降级：指把当前拥有的写锁，直接再获取到读锁，随后释放先前拥有的写锁的过程。

```java
readLock.unlock();//锁降级从写锁获取到开始
writeLock.lock();
try {
	if(!update){
	//准备数据的流程（略）
	update=true;
	readLock.lock();
} finally {
	writeLock.unlock();//锁降级完成，写锁降级为读锁
```

> LockSupport 定义了一组的公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能， 也成为构建同步组件的基础工具。
![](http://img.070077.xyz/202204271023067.png)

## Condition 接口
`Condition` 是定义了等待／通知两种类型的方法，当前线程调用这些方法时，需要提前获取对象关联的锁 （Condition 是依赖 Lock 对象的）。

![](http://img.070077.xyz/202204271758242.png)
![](http://img.070077.xyz/202204271759665.png)

ConditionObject 是 AQS 的内部类，其实现包括：等待队列 、等待和通知。
- 等待队列

![](http://img.070077.xyz/202204271812076.png)
- 等待

`await`方法会将当前线程构造成节点并加入等待队列中，然后释放同步状态，唤醒同步队列中的后继节点，然后当前线程会进入等待状态。

- 通知

`signal`方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中并使用 LockSupport 唤醒节点中的线程。

![](http://img.070077.xyz/202204271951154.png)


# 并发容器

## ConcurrentHashMap

-   JDK1.7 HashMap线程不安全体现在：循环链表、数据丢失。（此时的HashMap进行扩容（when `loadFactor` > 0.75）时，使用**头插法**进行迁移。）
-   JDK1.8 HashMap线程不安全体现在：数据覆盖。（并发put）

> `HashTable` 容器使用 synchronized 来保证线程安全，但在线程竞争激烈的情况下效率非常低下。 因为所有访问 HashTable的线程都必须竞争同一把锁，易进入阻塞或轮询状态 。 

ConcurrentHashMap容器使用锁分段技术，内含多把锁，每一把锁用于锁容器其中一部分数据。首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问 。
![](http://img.070077.xyz/202204272304694.png)

其初始化方法是通过 `initialCapacity` 、`loadFactor`（每个 segment 的负载因子） 和 `concurrencyLevel`等几个参数来初始化 segment 数组（大于或等于 concurrencyLevel 的最小的2幂次作为长度 。）、 段偏移量 segmentShift 、 段掩码 segmentMask 和每个segment 里的HashEntry 数组。

- `get`方法
  
  get 方法里将要使用的共享变量都定义成 volatile 类型，因此该方法不需加锁。

- `put`方法

会加锁。插入元素前会先判断 Segment 里的 HashEntry 数组是否超过容量。在扩容的时候，首先会创建一个容量是原来两倍的数组，然后将原数组里的元素进行再散列后插入到新的数组里。为了高效，ConcurrentHashMap 不会对整个容器进行扩容，而只对某个 segment 进行扩容。

- `size`方法

尝试 2 次通过不锁住 Segment 的方式来统计各个 Segment大小，如果统计的过程中，容器的 count 发生了变化（写元素都会将变量 modCount 进行加1），则再采用加锁的方式来统计所有Segment 的大小 。

## ConcurrentLinkedQueue

![](http://img.070077.xyz/202204272335337.png)


- 通过CAS算法入队

使用 hops 变量来控制并减少 tail 节点的更新频率，并不是每次节点入队后都将 tail 节点更新成尾节点，而是当 tail 节点和尾节点的距离大于等于 HOPS 的值 (默认等于1) 时才更新 tail 节点.
```java
Node<E> t=tail;
//p用来表示队列的尾节点，默认情况下等于tail节点。
Node<E> p=t;
for (int hops = 0; ; hops++) {
	//获得p节点的下一个节点。
	Node<E> next=succ(p);
	//next节点不为空，说明p不是尾节点，需要更新p后在将它指向next节点
	if (next != null) {
		//循环了两次及其以上，并且当前节点还是不等于尾节点
		if (hops > HOPS && t != tail )
			continue retry;
		p=next;
	}
	//如果p是尾节点，则设置p节点的next节点为入队节点。
	else if (p.casNext(null,n)){
	/*如果tail节点有大于等于1个next节点，则将入队节点设置成tail节点，
	更新失败了也没关系，因为失败了表示有其他线程成功更新了tail节点*/
		if(hops>=HOPS){
			casTail(t,n);//更新tail节点，允许失败
		return true;
	}
	//p有next节点，表示p的next节点是尾节点，则重新设置p节点
	else 
		p=succ(p);
}//...
```

- 出队也是类似的

首先获取头节点的元素，然后判断头节点元素是否为空，如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走，如果不为空，则使用 CAS 的方式将头节点的引用设置成 null，如果 CAS 成功，直接返回头节点的元素；如果不成功，表示另外一个线程已经进行了一次出队操作更新了 head 节点，导致元素发生了变化， 需要重新获取头节点 。

## 阻塞队列

阻塞队列(BlockingQueue)是一个支持两个附加操作的队列：
1) 支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满 。
2) 支持阻塞的移除方法 ：意思是在队列为空时，获取元素的线程会等待队列变为非空 。

![](http://img.070077.xyz/202204290209510.png)

特色DelayQueue 非常有用。比如：
- 缓存系统的设计：可以用 DelayQueue 保存缓存元素的有效期，使用一个线程循环查询 DelayQueue, 一旦能从 DelayQueue 中获取元素时，表示缓存有效期到了 。
- 定时任务调度 ：使用 DelayQueue 保存当天将会执行的任务和执行时间，一旦从 DelayQueue 中 获取 到任务就开始执行。如 `TimerQueue`

LinkedTransferQueue 是一个由链表结构组成的无界阻塞 TransferQueue 队列 。 相对其他阻塞队列多了 tryTransfer 和 transfer 方法 。

- 如果当前有消费者正在等待接收元素（ take/带时间限制的poll），transfer 方法可以把生产者传入的元素立刻 transfer 给消费者 。 如果没有消费者在等待接收元素 ，transfer 方法会将元素存放在队列的 tail 节点，并等到该元素被消费者消费了才返回.
-  tryTransfer 方法是用来试探生产者传入的元素是否能直接传给消费者，立即返回等待接收状态。

### 阻塞队列的实现
（ 通知模式 ）

ABQ使用了`Condition`，当往队列里插入一个元素时，如果队列不可用 ，那么通过`LockSupport.park(this)` 阻塞生产者。

## Fork/Join框架

![](http://img.070077.xyz/202204280027981.png)

工作窃取 (work-stealing) 算法是指某个线程从其他队列"窃取"任务来执行，充分利用线程进行并行计算，减少线程间竞争。但如双端队列里只有一个任务的情况，该算法消耗更多系统资源。

Fork/Join 使用两个类来完成分治和合并 。
1. ForkJoinTask : 我们要使用 ForkJoin 框架，必须首先创建一个 ForkJoin 任务 。它提供在任务中执行 fork(）和 join(）操作的机制 。 通常情况下，我们不需要直接继承 ForkJoinTask类，只需要继承它的子类RecursiveAction(用于没有返回结果的任务)/Recursive Task(有返回结果的任务）
2. ForkJoinPool : ForkJoinTask 需要通过 ForkJoinPool 来执行 。

任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部 。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务 。

ForkJoinTask 主要是compute 方法，在这个方法里，首先需要判断任务是否足够小，如果足够小就直接执行任务，不再分治。否则分割。

```java
@Override
protected Integer compute () {
	int sum = 0;
	boolean canCompute=(end-start)<=THRESHOLD;
	if(canCompute){
	//如果任务足够小就直接计算任务
	} else {//如果任务大，就分裂成两个子任务计算
		int middle=(start+end)/2;
		CountTask leftTask=new CountTask(start,middle);
		CountTask rightTask=new CountTask(middle+1,end);
		//执行子任务
		leftTask.fork();
		rightTask . fork () ;
		//等待子任务执行完，并得到其结果
		int leftResult=leftTask.join();
		int rightResult=rightTask.join();
		//合并子任务
		sum=leftResult+rightResult;
	}
	return sum;
```
.
### 实现原理

- `fork`：异步执行`((ForkJoinWorkerThread) Thread.current Thread ()). pushTask(this)`

- `join`：调用了 `doJoin`方法，得到当前任务的状态：已完成(NORMAL)、被取消 (CANCELLED ) 、信号 (SIGNAL) 和出现异常 (EXCEPTIONAL) 。

# Atomic包

大多数的原子类，本质上都是一个Unsafe和一个volatile变量的包装类。Unsafe 提供了 3 种 CAS 方法：`compareAndSwapObject` 、`compareAndSwapInt` 和 `compareAndSwapLong`.

>  原子更新基本类型类

- AtomicBoolean/Atomiclnteger/AtomicLong 人如其名，方法类似，以`AtomicLong`为例：
  - int addAndGet ( int delta ) ：以 原子方式将输入的数值与实例中的值相加 ，并返回结果。
  - boolean compareAndSet (int expect, int update) ：如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。
  -  int getAndlncrement(）：以原子方式将当前值加1, 返回自增**前**的值 。
  - void lazySet ( int newValue )：最终会设置成 newValue, 使用 lazySet 设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
  - int getAndSet (int newValue ) ：以原子方式设置为 newValue 值，并返回旧值 。

> 还有原子更新数组、引用类型、字段类的方法，略。


# 并发工具类

`CountDownLatch` 、`CyclicBarrier`和 `Semaphore` 工具类提供了一种并发流程控制的手段，`Exchanger` 工具类则提供了在线程间交换数据的一种手段 。

## 等待多个线程完成的 CountDownLatch

`CountDownLatch` 的构造函数接收一个 int 类型的参数作为计数器，如果你想等待 N 个小任务线程完成，这里就传入 N 。当我们调用 CountDownLatch 的 countDown 方法时，N--。用在多个线程时，只需要把这个 CountDownLatch 的引用传递到线程里即可 。

`CountDownLatch` 的 await 方法会阻塞当前线程，直到 N 变成零。可以使用另外一个带指定时间的 `await (long time, Time Unit unit)` ，等待特定时间后，就会不再阻塞当前线程。
类似于join/notify.

## 同步屏障 CyclicBarrier

让一组线程到达一个屏障（同步点）时被阻塞，直到最后一个线程到达屏障时，所有被屏障拦截的线程才会继续运行 。常用于多线程计算。

CyclicBarrier 默认的构造方法是 CyclicBarrier (int parties)，其参数表示屏障拦截的线程数，每个线程调用 await 方法告诉 CyclicBarrier 我巳经到达了屏障，然后当前线程被阻塞。还有`CyclicBarrier (int parties, Runnable barrierAction) `，用于在线程到达屏障时，优先执行 barrierAction。
其计数器可以使用 reset 方法重置。还有`getNumberWaiting`等，比上者强大。

## 控制井发线程数的 Semaphore

Semaphore （信号量）是用来控制同时访问特定资源的线程数最，它通过协调各个线程，以保证合理的使用公共资源，可用于流量控制。
构造方法 `Semaphore (int permits)` 接受一个整型的数字，表示可用的许可证数，也就是最大并发数。
首先线程使用 Semaphore 的 acquire 方法获取一个许可证，使用完之后调用 release 方法归还许可证。 还可以用 tryAcquire() 方法尝试获取许可证 。

## 线程间交换数据的 Exchanger

Exchanger 用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以通过 exchange 方法交换数据。如果第一个线程先执行 exchange()方法，它会一直等待第二个线程也执行 exchange 方法，即两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。常用于：遗传算法、校对。
可以使用 exchange (V x, longtimeout, TimeUnit unit) 设置最大等待时长 。

# 实例

- 生产者-消费者模式：生产者就是生产数据的线程，消费者就是消费数据的线程。通过阻塞队列来进行通信。
- 异步任务池
![](http://img.070077.xyz/202204290239954.png)

---
参考：
《Java并发编程的艺术》