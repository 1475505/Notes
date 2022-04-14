本章介绍 Java 并发包中相关的 API 和组件，以及这些 API 和组件的使用方式和实现细节。

# 锁

## Lock接口

Lock 接口出现之前，Java 程序是靠 synchronized 关键字实现锁功能的，而 Java SE 5 之后，并发包中新增了 Lock 接口（以及相关实现类）用来显式实现锁功能。

```java
Lock lock = new ReentrantLock();
lock.lock();
try {...} finally {lock.unlock();}
```

![](http://img.070077.xyz/202203290034087.png)

## 队列同步器

`AbstractQueuedSynchronizer` （简称同步器），是用来构建锁或者其他同步组件的基础框架，它使用了一个 int 成员变量表示同步状态，通过内置的 FIFO 队列来完成资源获取线程的排队工作。子类通过**继承**同步器并实现它的抽象方法来管理同步状态，使用同步器提供的 3 个方法 getState 、setState(int newState) 和 compareAndSetState(int expect, int update) 控制状态。



