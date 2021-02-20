## 介绍

当调用await()方法后，当前线程会释放锁并在此等待

synchronized与wait()和nitofy()/notifyAll()方法相结合可以实现等待/通知模型，ReentrantLock同样可以，但是需要借助Condition，且Condition有更好的灵活性，具体体现在：

1、一个Lock里面可以创建多个Condition实例，实现多路通知

2、notify()方法进行通知时，被通知的线程时Java虚拟机随机选择的，但是ReentrantLock结合Condition可以实现有选择性地通知，这是非常重要的

await()和signal()之前，必须要先lock()获得锁，使用完毕在finally中unlock()释放锁

**总结：**

1. ReentrantLock可以实现公平锁和非公平锁
2. ReentrantLock默认实现的是非公平锁
3. ReentrantLock的获取锁和释放锁必须成对出现，锁了几次，也要释放几次
4. 释放锁的操作必须放在finally中执行
5. lockInterruptibly()实例方法可以相应线程的中断方法，调用线程的interrupt()方法时，lockInterruptibly()方法会触发 `InterruptedException`异常
6. 关于 `InterruptedException`异常说一下，看到方法声明上带有 `throwsInterruptedException`，表示该方法可以相应线程中断，调用线程的interrupt()方法时，这些方法会触发 `InterruptedException`异常，触发InterruptedException时，线程的中断中断状态会被清除。所以如果程序由于调用 `interrupt()`方法而触发 `InterruptedException`异常，线程的标志由默认的false变为ture，然后又变为false
7. 实例方法tryLock()获会尝试获取锁，会立即返回，返回值表示是否获取成功
8. 实例方法tryLock(long timeout, TimeUnit unit)会在指定的时间内尝试获取锁，指定的时间内是否能够获取锁，都会返回，返回值表示是否获取锁成功，该方法会响应线程的中断

## 流程

当使用Condition.await()方法时，需要先获取Condition对象关联的ReentrantLock的锁，在Condition.await()方法被调用时，当前线程会释放这个锁，并且当前线程会进行等待（处于阻塞状态）。在signal()方法被调用后，系统会从Condition对象的等待队列中唤醒一个线程，一旦线程被唤醒，被唤醒的线程会尝试重新获取锁，一旦获取成功，就可以继续执行了。因此，在signal被调用后，一般需要释放相关的锁，让给其他被唤醒的线程，让他可以继续执行。

## **Condition接口原理**

Condition是一种广义上的条件队列。他为线程提供了一种更为灵活的等待/通知模式，线程在调用await方法后执行挂起操作，直到线程等待的某个条件为真时才会被唤醒。

Condition是AQS的内部类。每个Condition对象都包含一个队列(等待队列)。等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在Condition对象上等待的线程，如果一个线程调用了Condition.await()方法，那么该线程将会释放锁、构造成节点加入等待队列并进入等待状态。等待队列的基本结构如下所示。

![img](https://www.yht7.com/upload/image/20200605/20206595757657.png)



### await

当前线程调用await()方法，将会以当前线程构造成一个节点（Node），并将节点加入到该队列的尾部。

首先将当前线程新建一个节点同时加入到条件队列中，然后释放当前线程持有的同步状态。然后则是不断检测该节点代表的线程释放出现在CLH同步队列中（收到signal信号之后就会在AQS队列中检测到），如果不存在则一直挂起，否则参与竞争同步状态。

### 通知

调用Condition的signal()方法，将会唤醒在等待队列中等待最长时间的节点（条件队列里的首节点），在唤醒节点前，会将节点移到CLH同步队列中。

整个通知的流程如下：

1. 判断当前线程是否已经获取了锁，如果没有获取则直接抛出异常，因为获取锁为通知的前置条件。
2. 如果线程已经获取了锁，则将唤醒条件队列的首节点
3. 唤醒首节点是先将条件队列中的头节点移出，然后调用AQS的enq(Node node)方法将其安全地移到CLH同步队列中
4. 最后判断如果该节点的同步状态是否为Cancel，或者修改状态为Signal失败时，则直接调用LockSupport唤醒该节点的线程。

### 总结

一个线程获取锁后，通过调用Condition的await()方法，会将当前线程先加入到条件队列中，然后释放锁，最后通过isOnSyncQueue(Node node)方法不断自检看节点是否已经在CLH同步队列了，如果是则尝试获取锁，否则一直挂起。当线程调用signal()方法后，程序首先检查当前线程是否获取了锁，然后通过doSignal(Node first)方法唤醒CLH同步队列的首节点。被唤醒的线程，将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态。



## 参考

https://blog.csdn.net/chenssy/article/details/69279356