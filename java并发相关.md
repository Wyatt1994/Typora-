## 从synchronized 到CAS 和 AQS彻底弄懂Java各种并发锁

https://www.jianshu.com/p/8c1c047c9169

## 并发编程

### CAS

CAS指令在Intel CPU上称为CMPXCHG指令，**它的作用是将指定内存地址的内容与所给的某个值相比，如果相等，则将其内容替换为指令中提供的新值，如果不相等，则更新失败。**这一比较并交换的操作是原子的，不可以被中断。初一看，CAS也包含了读取、比较 (这也是种操作)和写入这三个操作，和之前的i++并没有太大区别，是的，的确在操作上没有区别，但**CAS是通过硬件命令保证了原子性**，而i++没有，且硬件级别的原子性比i++这样高级语言的软件级别的运行速度要快地多。虽然CAS也包含了多个操作，但其的运算是固定的(就是个比较)，这样的锁定性能开销很小。

从内存领域来说这是**乐观锁**，因为它在对共享变量更新之前会先比较当前值是否与更新前的值一致，如果是，则更新，如果不是，则无限循环执行(称为自旋)，直到当前值与更新前的值一致为止，才执行更新。

  简单的来说，**CAS**有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则返回V。这是一种乐观锁的思路，它相信在它修改之前，没有其它线程去修改它；而Synchronized是一种悲观锁，它认为在它修改之前，一定会有其它线程去修改它，悲观锁效率很低。下面来看一下AtomicInteger是如何利用CAS实现原子性操作的。

#### **AtomicInteger**

声明了一个**volatile**变量value，我们知道volatile保证了变量的内存可见性，也就是所有工作线程中同一时刻都可以得到一致的值。**Unsafe**是CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe，它提供了硬件级别的原子操作。

```
public final int addAndGet(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
    }


    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
            //该方法为本地方法，有四个参数，分别代表：对象、对象的地址、预期值、修改值
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    } 
```



#### CAS优点

通过硬件指令实现了非阻塞式的原子操作，锁定性能开销很小。

#### CAS缺点：

**循环时间太长**

如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。

**只能保证一个共享变量原子操作**

看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。例如读写锁中state的高地位

**ABA问题**(即在更新前的值是A，但在操作过程中被其他线程更新为B，又更新为 A)，

这时当前线程认为是可以执行的，其实是发生了不一致现象，如果这种不一致对程序有影响(真正有这种影响的场景很少，除非是在变量操作过程中以此变量为标识位做一些其他的事，比如初始化配置)，则需要使用AtomicStampedReference(除了对更新前的原值进行比较，也需要用更新前的 stamp标志位来进行比较)。对于ABA问题其**解决方案是加上版本号**，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。**AtomicStampedReference**解决了这个问题。

#### 总结： 

可以用CAS在无锁的情况下实现原子操作，但要明确应用场合，非常简单的操作且又不想引入锁可以考虑使用CAS操作，当想要非阻塞地完成某一操作也可以考虑CAS。不推荐在复杂操作中引入CAS，会使程序可读性变差，且难以测试，同时会出现ABA问题

## Synchronized锁的优化

**锁粗化（Lock Coarsening）：**也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。例如**多个连续的StringBuffer.append操作。**

**锁消除（Lock Elimination）：**通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过**逃逸分析**也可以在线程本地Stack上进行对象空间的分配（同时还可以减少Heap上的垃圾收集开销）。例如StringBuffer()的，在1.5前，字符串拼接底层通过StringBuffer()实现，这种情况是**不存在线程安全问题的，所以可以进行锁消除**。而1.5之后是通过StringBuilder()实现。

**轻量级锁（Lightweight Locking）：**这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态（即单线程执行环境），在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒（具体处理步骤下面详细讨论）。

**偏向锁（Biased Locking）：**是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟（可参考这篇[文章](https://blogs.oracle.com/dave/entry/biased_locking_in_hotspot)）。

**适应性自旋（Adaptive Spinning）：**当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁（mutex semaphore）前会进入忙等待（Spinning）然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore（即互斥锁）进入到阻塞状态。

## Volatile

任何进程在读取的时候，都会清空本**进程里面持有的共享变量的值**，强制从主存里面获取； 
任何进程在写入完毕的时候，都会**强制将共享变量的值写会主存**。 
volatile 会干预**指令重排**。 
volatile 实现了JMM规范的 **happen-before** 原则。

volatile只能保证**直接赋值具有原子性**，而i++这种多步操作时**不能保证**的，因为当有一个线程去读取主内存的过程中获取c的值，并拷贝一份放入自己的工作内存中，在对i进行+1操作的时候**线程阻塞**了（各种阻塞情况），那么这个时候有别的线程进入读取c的值，因为有一个线程阻塞就导致该线程无法体现出可见性，导致别的线程的**工作内存不会失效**，那么它还是从主内存中读取c的值，也会正常的+1操作。如此便导致了结果是小于等于1000的。



总结起来：

①使得变量更新变得具有可见性，只要被volatile修饰的变量的赋值**一旦变化就会通知到其他线程**，如果其他线程的工作内存中存在这个同一个变量拷贝副本，那么其他线程会放弃这个副本中变量的值，重新去主内存中获取

②产生了内存屏障，防止指令进行了**重排序**

## AQS

AQS，AbstractQueuedSynchronizer，即队列同步器。它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等）.

在基于AQS构建的同步器中，只能**在一个时刻发生阻塞**，从而降低上下文切换的开销，提高了吞吐量。因为AQS通过内置的**FIFO同步队列**来完成资源获取线程的排队工作，如果当前线程获取同步状态失败（锁）时，AQS则会将当前线程以及等待状态等信息构造成一个**节点**（Node）并将其加入**CLH同步队列**，同时会阻塞当前线程，当同步状态释放时，则会把节点中的线程唤醒（若为**公平锁**，则唤醒**首节点**），使其再次尝试获取同步状态。

**CLH同步队列**是一个FIFO**双向队列**，AQS依赖它来完成**同步状态**的管理。当有一个线程获取锁失败，会进行入队操作，其中使用了无限循环的方式调用CAS方法compareAndSetTail(Node expect, Node update)保证入队的线程安全和成功添加。当getState()方法值为0时，则可以出队，出队时不需要CAS，因为只会唤醒特定的某个节点（**公平锁则唤醒首节点**）

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。AQS使用一个int类型的成员变量**state**来表示同步状态，当state>0时表示已经获取了锁，当state = 0时表示释放了锁。



https://blog.csdn.net/ym123456677/article/details/80381354

**独占式同步状态获取流程，也就是acquire(int arg)方法调用流程**



![img](https://img-blog.csdn.net/20180520130953567?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ltMTIzNDU2Njc3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**总结：**在获取同步状态时，同步器维护一个同步队列，**获取状态失败的线程都会被加入到队列中并在队列中进行自旋**；移出队列（或停止自旋）的条件是前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器调用tryRelease(int arg)方法释放同步状态，然后唤醒**头节点的后继节点**。



## ReentrantLock

https://blog.csdn.net/Luxia_24/article/details/52403033

![è¿éåå¾çæè¿°](http://hi.csdn.net/attachment/201107/29/0_13119022769n5R.gif)



前面提到ReentrantLock提供了比synchronized更加灵活和强大的锁机制，那么它的灵活和强大之处在哪里呢？他们之间又有什么相异之处呢？**CAS+volatile**

### 获取锁

**ReentrantLock**当一个线程要获取锁时，直接**进行一次快速获取锁**，尝试compareAndSetState()，即CAS，若失败则acquire(1)，其中会判断一次调用相应的公平/非公平的tryAcquire()方法（需要实现的抽象方法）是否成功且同时将其入队CLH，。在tryAcquire()方法中，会再调用getState()进行**获取锁**。

```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

  以下方法主要逻辑：首先判断同步状态state == 0 ?，如果是表示该锁还没有被线程持有，直接通过CAS获取同步状态，如果成功返回true。如果state != 0，则判断当前线程是否为获取锁的线程，如果是则获取锁，成功返回true。成功获取锁的线程再次获取锁，这是增加了同步状态state。

```
protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }

    final boolean nonfairTryAcquire(int acquires) {
        //当前线程
        final Thread current = Thread.currentThread();
        //获取同步状态
        int c = getState();
        //state == 0,表示没有该锁处于空闲状态
        if (c == 0) {
            //获取锁成功，设置为当前线程所有
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        //线程重入
        //判断锁持有的线程是否为当前线程
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    } 
```

 

### 释放锁

获取同步锁后，使用完毕则需要释放锁，ReentrantLock提供了unlock释放锁：

```
    public void unlock() {
        sync.release(1);
    }
```

unlock内部使用AQS的模板方法的release(int arg)释放锁，release(int arg)是在AQS中定义的：

```
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
            	//从CLH队列中唤醒一个线程，注：无论公平非公平，释放锁唤醒线程都是默认唤醒当前线程的下一个节点，若节点为空（取消），则从tail队尾节点反向查找，直到找到一个离当前线程最近的非空节点
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

与获取同步状态的acquire(int arg)方法相似，释放同步状态的tryRelease(int arg)同样是需要自定义同步组件自己实现：

```
    protected final boolean tryRelease(int releases) {
        //减掉releases
        int c = getState() - releases;
        //如果释放的不是持有锁的线程，抛出异常
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        //state == 0 表示已经释放完全了，其他线程可以获取同步状态了
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
```

只有当同步状态彻底释放后该方法才会返回true。当state == 0 时，则将锁持有线程设置为null，free= true，表示释放成功。**注意：**判断是否为持有锁的线程，猜测是因为unlock()方法一般是在finally语句块中，若线程执行中出现异常，则执行finally语句块的不是当前线程，而是JVM系统线程。



### 公平锁与非公平锁区别

默认采用非公平锁，非公平锁进入调用lock()方法后**直接快速获取锁**（CAS)，失败则进入acquire()方法，然后调用tryAcquire()指向的Sync的unfairTryAcquire()。而公平锁没有进行快速锁获取，而是直接进入acquire()，其中的tryAcquire()指向fairAync的tryAcquire()方法。其中的区别在于tryAcquire()方法的**hasQueuedPredecessors**()，**公平锁会判断当前线程是否有为队列头部节点，也就是取队列头部节点。 ****而非公平锁获取锁直接选择当前线程，强调随机抢占。非公平锁与公平锁的区别在于**新晋获取锁的进程会有多次机会去抢占锁。如果被加入了等待队列后则跟公平锁没有区别。**而在**释放锁**时，从CLH队列中唤醒一个线程，无论**公平非公平**，释放锁唤醒线程都是默认唤醒当前线程的**下一个节点**，若节点为空（取消），则从tail队尾节点**反向查找**，直到找到一个离当前线程**最近**的非空节点。

```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

```
//公平锁的tryAcquire()方法
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
    	//多了一个方法hasQueuedPredecessors()
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

public final boolean hasQueuedPredecessors() {
        
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

```

//非公平锁的TryAcquire（）方法
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

### Condition应用

一个线程获取锁后，通过调用Condition的await()方法，会将当前线程先加入到条件队列中，判断条件（例如消费者发现产品数为0，则调用await()，然后释放自己占有的锁）然后释放锁，最后通过isOnSyncQueue(Node node)方法不断自检看节点是否已经在CLH同步队列了（因为signal会唤醒使其进入CLH队列），如果是则尝试获取锁，否则直接阻塞挂起（调用**park()方法**）。当线程调用signal()方法后，程序首先检查当前线程是否获取了锁，然后通过doSignal(Node first)方法唤醒CLH同步队列的首节点。被唤醒的线程，将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态。

#### await()等待

```
 public final void await() throws InterruptedException {
        // 当前线程中断
        if (Thread.interrupted())
            throw new InterruptedException();
        //当前线程加入条件队列
        Node node = addConditionWaiter();
        //释放锁
        long savedState = fullyRelease(node);
        int interruptMode = 0;
        /**
         * 检测此节点的线程是否在同步队上，如果不在，则说明该线程还不具备竞争锁的资格，则继续等待
         * 直到检测到此节点在同步队列上
         */
        while (!isOnSyncQueue(node)) {
            //线程挂起
            LockSupport.park(this);
            //如果已经中断了，则退出
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        //竞争同步状态
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        //清理下条件队列中的不是在等待条件的节点
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
```

此段代码的逻辑是：首先将当前线程新建一个节点同时加入到条件队列中，然后释放当前线程持有的同步状态。然后则是**不断检测**该节点代表的线程释放出现在CLH同步队列中（**为什么不断检测？因为每次收到signal信号之后就会在AQS队列中检测到**），如果不存在则一直挂起，否则参与竞争同步状态。

加入条件队列（addConditionWaiter()）源码如下：

```
    private Node addConditionWaiter() {
        Node t = lastWaiter;    //尾节点
        //Node的节点状态如果不为CONDITION，则表示该节点不处于等待状态，需要清除节点
        if (t != null && t.waitStatus != Node.CONDITION) {
            //清除条件队列中所有状态不为Condition的节点
            unlinkCancelledWaiters();
            t = lastWaiter;
        }
        //当前线程新建节点，状态CONDITION
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        /**
         * 将该节点加入到条件队列中最后一个位置
         */
        if (t == null)
            firstWaiter = node;
        else
            t.nextWaiter = node;
        lastWaiter = node;
        return node;
    }
```

该方法主要是将当前线程加入到Condition条件队列中。当然在加入到尾节点之前会清楚所有状态不为Condition的节点。



#### signal()唤醒

调用Condition的signal()方法，将会唤醒在（条件队列）等待队列中等待最长时间的节点（条件队列里的首节点），在唤醒节点前，会将节点移到**CLH同步队列**中。

```
    public final void signal() {
        //检测当前线程是否为拥有锁的独
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        //头节点，唤醒条件队列中的第一个节点
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);    //唤醒
    }
```

该方法首先会判断当前线程是否已经获得了锁，这是前置条件。然后唤醒条件队列中的头节点。

doSignal(Node first)：唤醒头节点

```
    private void doSignal(Node first) {
        do {
            //修改头结点，完成旧头结点的移出工作
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                (first = firstWaiter) != null);
    }
```

doSignal(Node first)主要是做两件事：1.修改头节点，2.调用transferForSignal(Node first) 方法将节点移动到CLH同步队列中。transferForSignal(Node first)源码如下：

```
     final boolean transferForSignal(Node node) {
        //将该节点从状态CONDITION改变为初始状态0,
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        //将节点加入到syn队列中去，返回的是syn队列中node节点前面的一个节点
        Node p = enq(node);
        int ws = p.waitStatus;
        //如果结点p的状态为cancel 或者修改waitStatus失败，则直接唤醒
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```

**整个通知的流程如下：**

1. 判断当前线程是否已经获取了锁，如果没有获取则直接抛出异常，因为获取锁为通知的前置条件。
2. 如果线程已经获取了锁，则将唤醒**条件队列的首节点**
3. 唤醒首节点是**先将条件队列中的头节点移出，然后调用AQS的enq(Node node)方法将其安全地移到CLH同步队列中**
4. 最后判断如果该节点的同步状态是否为Cancel，或者修改状态为Signal失败时，则直接调用LockSupport唤醒该节点的线程。

### 与synchronized的区别：

首先他们肯定具有相同的功能和内存语义，ReentrantLock提供了比synchronized更加灵活和强大的锁机制。

1. 与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。
2. ReentrantLock还提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更加适合（以后会阐述Condition）。
3. ReentrantLock提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized则一旦进入锁请求要么成功要么阻塞，所以相比synchronized而言，ReentrantLock会不容易产生死锁些。
4. ReentrantLock支持更加灵活的同步代码块，但是使用synchronized时，只能在同一个synchronized块结构中获取和释放。注：ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。
5. ReentrantLock支持中断处理，且性能较synchronized会好些。

### CLH同步队列

https://mp.weixin.qq.com/s？__biz=MzUzMTA2NTU2Ng==&mid=2247483907&idx=1&sn=2a297b6a961368696ba7fe13e6f20007&chksm=fa497db2cd3ef4a493f523441346aab29a1f95658f49eabc744fbee56f76d1239eecd162ef66&scene=21#wechat_redirect

### 总结

**ReentrantLock获取锁和释放锁**的过程差异：获取锁lock()调用后会有一次快速获取锁CAS尝试，如果失败则调用**AQS的模板方法acquire()**，释放锁同样是调用AQS的release()方法，进行tryAcquire以及入队，通过通过**自旋**的方式不断获取同步状态，但也不会**一直自旋**，但因此需要在自旋的过程中则需要判断当前线程是否需要阻塞，即检查状态，其主要方法在acquireQueued()：

```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

```
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

检查状态的方法为acquireQueue()中的 shouldParkAfterFailedAcquire(Node pred, Node node) 方法，该方法主要**靠前驱节点判断当前线程是否应该被阻塞**。

1.如果前一个节点状态为signal，表示**前驱节点等待被唤醒**，则当前线程需被阻塞，直接放回true，SIGNAL状态的节点，释放锁后，会唤醒其后继节点。因此，此线程可以安全的阻塞（前驱节点释放锁时，会唤醒此线程)。

2.若前驱节点状态 > 0 ，则为Cancelled,表明该节点已经超时或者被中断了，需要从同步队列中删除。

3.如果前驱节点非SINNAL，非CANCELLED，则通过CAS的方式将其前驱节点设置为SINNAL，然后返回false，保持自旋。

总体看来，shouldParkAfterFailedAcquire就是靠**前继节点**判断当前线程是否应该被阻塞，如果前继节点处于CANCELLED状态，**则顺便删除这些节点重新构造队列。** 

若shouldParkAfterFailedAcquire()返回true，则需要被阻塞，则调用parkAndCheckInterrupt()方法阻塞当前线程：

```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //若返回true，则被当前线程被阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

```
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

需要阻塞或者唤醒一个线程的时候，AQS的都是使用**LockSupport**这个工具类来完成的。唤醒对应unparkSuccessor(h)方法。



**LockSupport是用来创建锁和其他同步类的基本线程阻塞原语**，内部的实现都是通过UNSAFE（sun.misc.Unsafe UNSAFE）来实现的

`public native void park(boolean var1, long var2);`

`public native void unpark(Object var1);`

两个都是**native本地方法**。Unsafe 是一个比较危险的类，主要是用于执行低级别、不安全的方法集合。尽管这个类和所有的方法都是公开的（public），但是这个类的使用仍然受限，你无法在自己的java程序中直接使用该类，因为只有授信的代码才能获得该类的实例。



其中tryAcquire中会有判断持有锁线程是否为当前线程，用于实现**锁可重入**，同时每重入一次，会state+1。而release()则是直接tryRelease，需要state-release，判定是否为当前线程，若不是抛出异常，否则将判定c是否为0，即设置无锁状态成功，需要锁持有线程setExclusiveOwnerThread设置为null，free= true，然后从CLH中指定某个线程唤醒。



## ReentrantReadWriteLock（读写锁）

https://mp.weixin.qq.com/s？__biz=MzUzMTA2NTU2Ng==&mid=2247484040&idx=1&sn=60633c2dc4814b26dc4b39bb2bb5d4dd&chksm=fa497d39cd3ef42f539cd0576c1a3575ee27307048248571e954f0ff21a5a9b1ddfab522c834&scene=21#wechat_redirect

读锁本质上是个共享锁。

但读锁对锁的获取做了很多优化，比如使用  firstReader  和 cachedHoldCounter 最**第一个读锁线程和最后一个读锁线程做优化，优化点主要在释放的时候对计数器的获取。**

同时，如果在获取读锁的过程中写锁被持有了，JUC 并没有让所有线程痴痴的等待，而是判断入如果获取读锁的线程是正巧是持有写锁的线程，那么当前线程就可以降级获取写锁，否则就会死锁了（**为什么死锁，当持有写锁的线程想获取读锁，但却无法降级，进入了等待队列，肯定会死锁**）。

还有一点就是性能上的优化，如果先释放写锁，再获取读锁，势必引起锁的争抢和线程上下文切换，影响性能。



读写锁ReentrantReadWriteLock实现**接口ReadWriteLock**，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

```
public interface ReadWriteLock {
	//返回读操作的锁
    Lock readLock();
	//返回写操作的锁
    Lock writeLock();
```

在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。但是读写锁ReentrantReadWriteLock内部维护着两个一对锁，需要用一个变量维护多种状态。所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，**高16为表示读，低16为表示写**。分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过为运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。

```
/** 返回共享锁（即读锁）  */
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
/** 返回排它锁（即写锁）  */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

### 主要特性

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。**读写锁最多支持65535个递归写入锁和65535个递归读取锁（包括重入）。**
3. 锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁

#### 写锁

写锁就是一个支持可重入的排他锁。**ReentrantReadWriteLock**与**ReentrantLock**一样，其锁主体依然是**Sync**，它的读锁、写锁都是依靠Sync来实现的，**但tryAcquire()方法都实现了**，而ReentrantLock的Sync抽象内部类则只实现了tryRealease()，需要Sync的子类nofairSync和fairSync单独进行实现tryAcquire()方法。所以ReentrantReadWriteLock实际上**只有一个锁**，只是在获取读取锁和写入锁的方式上不一样而已，它的读写锁其实就是两个类：ReadLock、writeLock，这两个类都是lock实现。

##### 获取写锁

在判断重入时多了一个c!=0和w==0**判断是否有无读锁**。若c!=0且w==0则表示**存在读锁**，因为要确保**写锁的操作对读锁是可见**的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此**只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。**

注：exclusiveCount(c)方法用于获取写锁的数量。获取数量时和00FF做&运算得到高16位的值。

**为啥要判定写锁超过最大范围呢？**

因为递归写锁的**重入次数**也有上限，即65535，故需要和MAX_COUNT值为1左移16位-1,即65535。

```
protected final boolean tryAcquire(int acquires) {
        Thread current = Thread.currentThread();
        //当前锁个数，包含读写锁
        int c = getState();
        //返回占有的写锁个数
        int w = exclusiveCount(c);
        if (c != 0) {
            //c != 0 && w == 0 表示存在读锁
            //当前线程不是已经获取写锁的线程
            if (w == 0 || current != getExclusiveOwnerThread())
                return false;
            //超出最大范围
            if (w + exclusiveCount(acquires) > MAX_COUNT)
                throw new Error("Maximum lock count exceeded");
            setState(c + acquires);
            return true;
        }
        //是否需要阻塞
        if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
            return false;
        //设置获取锁的线程为当前线程
        setExclusiveOwnerThread(current);
        return true;
    }
```

##### 释放写锁

WriteLock提供了unlock()方法释放写锁：

```
   public void unlock() {
        sync.release(1);
    }

	//AQS中的模板方法
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }  
```

写锁释放锁的整个过程和独占锁ReentrantLock相似，每次释放均是**减少写状态**，当写状态为0时表示 写锁已经完全释放了，从而等待的其他线程可以继续访问读写锁，获取同步状态，同时**此次写线程的修改对后续的线程可见**。

```
protected final boolean tryRelease(int releases) {
        //释放的线程不为锁的持有者
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int nextc = getState() - releases;
        //若写锁的新线程数为0，则将锁的持有者设置为null
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
            setExclusiveOwnerThread(null);
        setState(nextc);
        return free;
    }
```

写锁的释放最终还是会调用AQS的模板方法release(int arg)方法，该方法首先调用tryRelease(int arg)方法尝试释放锁，tryRelease(int arg)方法为读写锁内部类Sync中定义了。

#### 读锁

读锁为一个**可重入的共享锁**，它能够被**多个线程**同时持有，**在没有其他写线程访问时，读锁总是或获取成功。**

##### 获取读锁

读锁的获取可以通过ReadLock的lock()方法：

```
        public void lock() {
            sync.acquireShared(1);
        }
```

Sync的acquireShared(int arg**)定义在AQS中**：

```
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

tryAcqurireShared(int arg)**定义在Sync抽象内部类中**。尝试获取读同步状态，该方法主要用于获取共享式同步状态，定义在获取成功返回 >= 0的返回结果，否则返回 < 0 的返回结果。

```
protected final int tryAcquireShared(int unused) {
        //当前线程
        Thread current = Thread.currentThread();
        int c = getState();
        //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题，后续阐述
        if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
            return -1;
        //读锁
        int r = sharedCount(c);

​```
    /*
     * readerShouldBlock():读锁是否需要等待（公平锁原则）
     * r < MAX_COUNT：持有线程小于最大数（65535）
     * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
     */
    if (!readerShouldBlock() &&
            r < MAX_COUNT &&
            compareAndSetState(c, c + SHARED_UNIT)) {
        /*
         * holdCount部分
         *获取读锁时，首先判断是否是firstReader，即第一个获取读锁的线程，若是，则直接加1,否则将
         *获取上一个获取读锁的线程，即cachedHoldCounter，若不等于当前线程，则调用全局ThreadLocal
         *HoldCounter的get()获取当前线程的holdcounter(rh)并赋给cachedHoldCounter，同时若rh的
         *count为0，则将其加入readHold，同时计数加1
         */
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    //这个方法会根据“是否需要阻塞等待”，“读取锁的共享计数是否超过限制”等等进行处理。判断是否需要等待
    //根据公平锁和非公平锁单独判断，若为公平锁，则判断前驱CLH有无同步队列节点，若有则阻塞。若为
    //非公平锁，则无需判断。
    //如果不需要阻塞等待，并并且锁的共享计数没有超过限制，则通过CAS尝试获取锁，并返回1
    return fullTryAcquireShared(current);
}
​```


```

**为何要引入firstRead、firstReaderHoldCount？**

这是为了一个效率问题，firstReader是不会放入到readHolds中的，如果读锁仅有一个的情况下就会避免查找readHolds。

**为何要引入cachedHoldCounter？**

cachedHoldCounter为缓存的上一个读取线程。因为这样可以减少ThreadLocal.get()的次数，因为这也是一个耗时操作。需要说明的是这样HoldCounter绑定线程id而不绑定线程对象的原因是避免HoldCounter和ThreadLocal互相绑定而GC难以释放它们（尽管GC能够智能的发现这种引用而回收它们，但是这需要一定的代价），所以其实这样做只是为了帮助GC快速回收对象而已。

##### 释放读锁

与写锁相同，读锁也提供了unlock()释放读锁：

```
        public void unlock() {
            sync.releaseShared(1);
        }
```

unlcok()方法内部使用Sync的releaseShared(int arg)方法，该方法定义在AQS中：

```
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

调用tryReleaseShared(int arg)尝试释放读锁，该方法定义在读写锁的Sync内部类中：

```
    protected final boolean tryReleaseShared(int unused) {
        Thread current = Thread.currentThread();
        //如果想要释放锁的线程为第一个获取锁的线程
        if (firstReader == current) {
            //仅获取了一次，则需要将firstReader 设置null，否则 firstReaderHoldCount - 1
            if (firstReaderHoldCount == 1)
                firstReader = null;
            else
                firstReaderHoldCount--;
        }
        //获取rh对象，并更新“当前线程获取锁的信息”
        else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                rh = readHolds.get();
            int count = rh.count;
            if (count <= 1) {
                readHolds.remove();
                if (count <= 0)
                    throw unmatchedUnlockException();
            }
            --rh.count;
        }
        //CAS更新同步状态
        for (;;) {
            int c = getState();
            int nextc = c - SHARED_UNIT;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
```

#### 锁降级

锁降级就意味着**写锁是可以降级为读锁**的，但是需要遵循**先获取写锁、获取读锁在释放写锁**的次序。注意如果当前线程先获取写锁，然后释放写锁，再获取读锁这个过程不能称之为锁降级，锁降级一定要遵循那个次序。

在获取读锁的方法tryAcquireShared(int unused)中，有一段代码就是来判读锁降级的：

```
        int c = getState();
        //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题，后续阐述
        if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
            return -1;
        //读锁
        int r = sharedCount(c);
```

锁降级中读锁的获取释放为必要？肯定是必要的。试想，假如当前线程A不获取读锁而是直接释放了写锁，这个时候另外一个线程B获取了写锁，那么这个线程B对数据的修改是不会对当前线程A可见的。如果获取了读锁，则线程B在获取写锁过程中判断如果有读锁还没有释放则会被阻塞，只有当前线程A释放读锁后，线程B才会获取写锁成功。

**锁降级中读锁的获取释放为必要？**

肯定是必要的。试想，假如当前线程A不获取读锁而是直接释放了写锁，这个时候另外一个线程B获取了写锁，那么这个线程B对数据的修改是不会对当前线程A可见的。如果获取了读锁，则线程B在获取写锁过程中判断如果有读锁还没有释放则会被阻塞，只有当前线程A释放读锁后，线程B才会获取写锁成功。

#### 总结

##### **读锁写锁的差异**

**获取读锁**时，读锁为一个可重入的共享锁。必须**保证没有写锁**，且有65535的限制。同时还有个特点就是添加了一个**锁降级的判定**，即一个线程获取写锁后又是否又在尝试获取读锁。首先判断是否是firstReader，即第一个获取读锁的线程，若是，则直接加1,否则将获取上一个获取读锁的线程，即cachedHoldCounter，若不等于当前线程，则调用全局ThreadLocalHoldCounter的get()获取当前线程的holdcounter(rh)并赋给cachedHoldCounter，同时若rh的count为0，则将其加入readHold，同时计数加1。**释放读锁**时，也需要使用到firstReader和cachedHoldCounter。读锁中，最主要的就是**cachedHoldCounter**、ThreadLocalHoldCounter为每个线程维护的**holdcount**以及**firstReader**和**firstReaderHoldCounter**。

**获取写锁时，**跟重入锁ReentrantLock的主要区别是加入一个判定条件，即读锁是否存在，即必须要**等待读锁完全释放之后才能获取写锁**。因为要确保写锁的操作对读锁是可见的，若不这样，则获取读锁的其他线程无法感知。**写锁获取后，其他读、写线程均会被阻塞。**

**注**：是**允许一个线程获取写锁后再次获取读锁**，也因此需要在获取读锁时进行特殊处理，即某个线程获取读锁时，若存在写锁且这个线程并不是拥有写锁的线程，则直接返回-1。

```
		int c = getState();
        //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题
        if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
            return -1;
        //读锁
        int r = sharedCount(c);
```



## CountdownLatch

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfcsic1PHSEJVAwibzrUmNRgfibuycTibPcFKDFWfejExvdZyKIWbiaQ0EZlTEwLFs4PQ7FItqXYCibvKugw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



一般用做“发令枪”，即多个线程等待好之后同时开始，模拟并发。

https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247484300&idx=1&sn=fcdadc7aeebfd397731820a50bbf1374&chksm=fa497c3dcd3ef52b9645f2912e2674c03944d36a1e5638e42da7a30b928d85a51746682b1df7&scene=21#wechat_redirect

#### 总结

CountDownLatch内部通过**共享锁(跟读锁一样，使用AQS的模板方法，acquireShared和releaseShared)**实现。在创建CountDownLatch实例时，需要传递一个int型的参数：count，该参数为计数器的初始值，即将其**值赋给state（volatile变量）**也可以理解为该共享锁可以获取的总次数。当某个线程调用await()方法，程序首先判断count的值是否为0，如果不会0的话则会加入**同步队列**进行自旋（**通过for的死循环，当然也会判断是否需要被阻塞，即根据前一个节点的状态**）不断检查（**getState方法**）直到为0为止。当其他线程调用countDown()方法时，则执行释放共享锁状态，使count值 - 1。当在创建CountDownLatch时初始化的count参数，必须要有count线程调用countDown方法才会使计数器count等于0，锁才会释放，前面等待的线程才会继续运行。注意CountDownLatch**不能回滚重置。**

```
public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
//getstate！=0，则加入CLH同步队列，并自旋（死循环），并判断是否自旋线程是否需要被阻塞
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

countDownLatch.await() countDownLatch.countDown();

#### 开会应用示例

```
  public class CountDownLatchTest {
    private static CountDownLatch countDownLatch = new CountDownLatch(5);

    /**
     \* Boss线程，等待员工到达开会
     */
    static class BossThread extends Thread{
        @Override
        public void run() {
            System.out.println("Boss在会议室等待，总共有" + countDownLatch.getCount() + "个人开会...");
            try {
                //Boss等待
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("所有人都已经到齐了，开会吧...");
        }
    }

    //员工到达会议室
    static class EmpleoyeeThread  extends Thread{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "，到达会议室....");
            //员工到达会议室 count - 1
            countDownLatch.countDown();
        }
    }

    public static void main(String[] args){
        //Boss线程启动
        new BossThread().start();

        for(int i = 0 ; i < countDownLatch.getCount() ; i++){
            new EmpleoyeeThread().start();
        }
    }
}  
```



## Semaphore

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZffIthIUgWiaTkZibzFicNx5sM0jAhnzvqp1eJ4iaELUugOzVvepw6SSqMibAHJBmFCsyCMhibuZsicTuJdCQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



用做有限“**停车场**”问题，获取和释放跟countdownlatch类似,比如获取锁都调用A**QS的acquireSharedInterruptibly（）**方法，若getState（）大于0,则尝试获取，否则加入CLH同步队列自旋然后判定是否阻塞。默认采用**非公平模式**，即不需要判断当前线程是否位于**CLH同步队列列头**。释放锁类似。

semaphore.acquire()  semaphore.release()

```
public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }

protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        
//getstate！=0，则加入CLH同步队列，并自旋（死循环），并判断是否自旋线程是否需要被阻塞
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### 应用示例

停车为示例：

```
public class SemaphoreTest {

    static class Parking{
        //信号量
        private Semaphore semaphore;

        Parking(int count){
            semaphore = new Semaphore(count);
        }

        public void park(){
            try {
                //获取信号量
                semaphore.acquire();
                long time = (long) (Math.random() * 10);
                System.out.println(Thread.currentThread().getName() + "进入停车场，停车" + time + "秒..." );
                Thread.sleep(time);
                System.out.println(Thread.currentThread().getName() + "开出停车场...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
        }
    }


    static class Car extends Thread {
        Parking parking ;

        Car(Parking parking){
            this.parking = parking;
        }

        @Override
        public void run() {
            parking.park();     //进入停车场
        }
    }

    public static void main(String[] args){
        Parking parking = new Parking(3);

        for(int i = 0 ; i < 5 ; i++){
            new Car(parking).start();
        }
    }}
```

## CyclicBarrier

![img](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfcDQUAmYsQTrbFNOnj3TQLZWT1qI8OVz7qzyxVN2ycZbqxDFL8jbN6zyialGymibiaxubL25eiabIJpFQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



CyclicBarrier的内部是使用**重入锁ReentrantLock和Condition**。它有两个构造函数：

- CyclicBarrier(int parties)：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
- CyclicBarrier(int parties, Runnable barrierAction) ：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

CyclicBarrier试用与多线程结果合并的操作，用于**多线程计算数据**，最后合并计算结果的应用场景。比如我们需要统计多个Excel中的数据，然后等到一个总结果。我们可以通过多线程处理每一个Excel，执行完成后得到相应的结果，最后通过barrierAction来计算这些线程的计算结果，得到所有Excel的总和。



```
public int await() throws InterruptedException, BrokenBarrierException {
    try {
    
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

private int dowait(boolean timed, long nanos)
            throws InterruptedException, BrokenBarrierException,
            TimeoutException {
        //获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //分代
            final Generation g = generation;

            //当前generation“已损坏”，抛出BrokenBarrierException异常
            //抛出该异常一般都是某个线程在等待某个处于“断开”状态的CyclicBarrie
            if (g.broken)
                //当某个线程试图等待处于断开状态的 barrier 时，或者 barrier 进入断开状态而线程处于等待状态时，抛出该异常
                throw new BrokenBarrierException();

            //如果线程中断，终止CyclicBarrier
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }

            //进来一个线程 count - 1
            int index = --count;
            //count == 0 表示所有线程均已到位，触发Runnable任务
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    //触发任务
                    if (command != null)
                        command.run();
                    ranAction = true;
                    //唤醒所有等待线程，并更新generation
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }


            for (;;) {
                try {
                    //如果不是超时等待，则调用Condition.await()方法等待
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        //超时等待，调用Condition.awaitNanos()方法等待
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                //generation已经更新，返回index
                if (g != generation)
                    return index;

                //“超时等待”，并且时间已到,终止CyclicBarrier，并抛出异常
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            //释放锁
            lock.unlock();
        }
    }
```

cyclicBarrier.await()

####  应用实例

老板开会

```
 public class CyclicBarrierTest {
    private static CyclicBarrier cyclicBarrier;

    static class CyclicBarrierThread extends Thread{
        public void run() {
            System.out.println(Thread.currentThread().getName() + "到了");
            //等待
            try {
                cyclicBarrier.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        cyclicBarrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("人到齐了，开会吧....");
            }
        });

        for(int i = 0 ; i < 5 ; i++){
            new CyclicBarrierThread().start();
        }
    }
}  
```





## BlockingQueue

http://cmsblogs.com/?p=2122

ArrayBlockingQueue、LinkedBlockingQueue 之类使用了ReentranLock，本质还是使用AQS实现并发操作，再使用CAS对state状态的更改。而SynchronousQueue则没有使用AQS，而是直接使用CAS操作**首指针和尾指针来实现**。SynchronousQueue是无容量的，最大特点是没有锁住整个队列（利用CAS）因此若先put，一旦**没有消费者则丢弃，不会缓存**。而LinkedTransferQueue相当于有容量，若无数据，则能**缓存消费者消费请求**，即**预占模式，**一直等待生产者的数据。若**无消费者**，也能缓存生产者的数据（**必须使用transfer（）方法才能缓存，tryTransfer不会缓存**）。

JDK 8 中提供了七个阻塞队列可供使用：

- ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
- LinkedBlockingQueue ：一个由链表结构组成的无界阻塞队列。
- PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
- SynchronousQueue：一个不存储元素的阻塞队列。
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

### ArrayBlockingQueue

用数组实现**有边界**的阻塞队列,采用一个重入锁的两个condition,避免复杂化，对于性能而言应该不会有太大的提升。因此无法做到真正意义的消费者、生产者**并行执行**，也就是同时添加和获取。创建时必须**指定容量**。其构**造方法需要获取互斥锁，用于保证数组创建的可见性**，避免**重排序问题**，即引用赋值和初始化对象步骤错位。

### LinkedBlockingQueue

**单向链表**结构的有界阻塞队列。如果不指定容量**默认**为**Integer.MAX_VALUE**。通过putLock和takeLock**两个锁**进行同步，两个锁分别实例化notFull和notEmpty**两个Condtion**，用来协调多线程的存取动作，因此可以实现消费者生产者**同时执行**。其中某些方法(如**remove,toArray,toString,clear**等)的同步需要**同时获得这两个锁**，并且总是先putLock.lock紧接着takeLock.lock(在同一方法fullyLock中)，这样的顺序是为了避免可能出现的死锁情况

### LinkedBlockingDeque

LinkedBlockingDeque是一个有链表组成的**双向阻塞队列**，采用**一个重入锁的两个condition。**与前面的阻塞队列相比它支持从两端插入和移出元素。以first结尾的表示从对头操作，以last结尾的表示从对尾操作。

在初始化LinkedBlockingDeque时可以初始化队列的容量，用来防止其**再扩容时过渡膨胀**。另外双向阻塞队列可以运用在“工作窃取”模式中。

### PriorityBlockingQueue

PriorityBlockingQueue是支持优先级的无界队列。默认情况下采用自然顺序排序，当然也可以通过自定义**Comparator**来指定元素的排序顺序,不能保证同**优先级元素的顺序**。

PriorityBlockingQueue内部采用**二叉堆**的实现方式，整个处理过程并不是特别复杂。**添加操作**则是不断“上冒”，而**删除操作**则是不断“下掉”。例如**添加操作**，即元素**末尾插入元素**，然后不断跟父节点比较替换直到不能移动为止。而**删除操作**，都是删除**第一个元素**，即arr[0]

内部仍然采用**可重入锁ReentrantLock**来实现同步机制，但是这里**只有一个notEmpty的Condition**，了解了ArrayBlockingQueue我们知道它定义了两个Condition，之类为何只有一个呢？原因就在于PriorityBlockingQueue是一个**无界队列**，**插入总是会成功**，除非消耗尽了资源导致服务器挂。

默认容量为**11**，默认最大容量为**MAX.Interger-8**。扩容采用**翻倍**原则，底层就是一个Object[]数组

### DelayQueue

DelayQueue是一个支持**延时操作的无界阻塞队列**。列头的元素是最先“到期”的元素，如果**队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行**。也就是说只有在**延迟期满**时才能够从队列中去元素。

它主要运用于如下场景：

- 缓存系统的设计：缓存是有一定的时效性的，可以用DelayQueue保存缓存的有效期，然后利用一个线程查询DelayQueue，如果取到元素就证明该缓存已经失效了。
- 定时任务的调度：DelayQueue保存当天将要执行的任务和执行时间，一旦取到元素（任务），就执行该任务。

DelayQueue采用**支持优先级的PriorityQueue**来实现，但是队列中的元素必须要实现Delayed接口，Delayed接口用来标记那些应该在给定延迟时间之后执行的对象，该接口提供了getDelay()方法返回元素节点的剩余时间。同时，元素也必须要实现compareTo()方法，compareTo()方法需要提供与getDelay()方法一致的排序。

### SynchronousQueue

SynchronousQueue是一个神奇的队列，他是一个不存储元素的阻塞队列，也就是说他的**每一个put操作都需要等待一个take操作**，若有**多余的take和put，则会一直阻塞，每来一个配对的线程，则执行一次，直到全部匹配完成，才会结束线程**。有点儿像[Exchanger](http://cmsblogs.com/?p=2269)，类似于生产者和消费者进行交换，通过**CAS来操作首尾指针**实现。

队列本身不存储任何元素，所以非常适用于传递性场景，两者直接进行对接。其吞吐量会高于ArrayBlockingQueue和LinkedBlockingQueue。

SynchronousQueue支持公平和非公平的访问策略，在默认情况下采用非公平性，也可以通过构造函数来设置为公平性。

SynchronousQueue的实现核心为Transferer接口，该接口有**TransferQueue**和**TransferStack**两个实现类，分别对应着公**平策略**和**非公平策略**。接口Transferer有一个tranfer()方法，该方法定义了转移数据，如果e != null，相当于将一个数据交给消费者，如果e == null，则相当于从一个生产者接收一个消费者交出的数据。

### LinkedTransferQueue

含图解析：https://www.cnblogs.com/duanxz/p/3252267.html

**LinkedTransferQueue**就是队列即使不存在数据节点，也会存放请求节点。当一个线程试图放入一个数据节点，正好遇到一个请求数据的结点，会**立刻匹配并移除该数据节点**，对于请求节点入队列也是一样的。Blocking Dual Queues**阻塞所有未匹配的线程**，直到有匹配的线程出现。

LinkedTransferQueue是一个由链表组成的的无界阻塞队列，该队列是一个相当牛逼的队列：它是ConcurrentLinkedQueue、SynchronousQueue (公平模式下)、无界的LinkedBlockingQueues等的超集。

与其他BlockingQueue相比，他多实现了一个接口TransferQueue，该接口是对BlockingQueue的一种补充，多了tryTranfer()和transfer()两类方法：

- tranfer()：若当前存在一个正在等待获取的消费者线程，即立刻移交之。 否则，会插入当前元素e到队列尾部，并且等待进入阻塞状态，到有消费者线程取走该元素
- tryTranfer()： 若当前存在一个正在等待获取的消费者线程（使用take()或者poll()函数），使用该方法会即刻转移/传输对象元素e；若不存在，则返回false，**并且不进入队列**。这是一个不阻塞的操作

## ConcurrentHashMap

在jdk1.8中ConcurrentHashMap主要是采用了**3个CAS操作**实现线程安全的

主要设计上的变化有以下几点:

1. 不采用segment而采用node，锁住node来实现减小锁粒度。
2. 设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。
3. 使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。
4. sizeCtl的不同值来代表不同含义，起到了控制的作用。

至于为什么JDK8中使用synchronized而不是ReentrantLock，我猜是因为JDK8中对synchronized有了足够的优化吧。

http://www.importnew.com/22007.html

https://blog.csdn.net/u012403290?t=1

http://cmsblogs.com/?p=2283

**transfer方法（扩容方法）** 

数组中（桶中）总共分为3种存储情况：空，链表头，TreeBin头 
①遍历原来的数组（原table），如果数组中某个值为空，则直接放置一个forwordingNode（上篇博文介绍过）。 
②如果数组中某个值不为空，而是一个链表头结点，那么就对这个链表进行拆分为两个链表，存储到nextTable对应的两个位置。 

③如果数组中某个值不为空，而是一个TreeBin头结点，那么这个地方就存储的是红黑树的结构，这样一来，处理就会变得相对比较复杂，就需要先判断需不需要把**树转换为链表**（**小于等于6**），做完一系列的处理，然后把对应的结果存储在nextTable的对应两个位置。

**PUT方法** 

①先传入一个k和v的键值对，不可为空（HashMap是可以为空的），如果为空就直接报错。 
②接着去判断table是否为空，如果为空就进入初始化阶段。 
③如果判断数组中某个指定的桶是空的，那就直接把键值对插入到这个桶中作为头节点，而且这个操作不用加锁**(CAS操作赋值)。** 
④如果这个要插入的桶中的hash值为-1，也就是MOVED状态（也就是这个节点是forwordingNode），那就是说明有线程正在进行扩容操作，那么当前线程就进**入协助扩容**阶段。helpTransfer()方法为**协助扩容方法**，当调用该方法的时候，nextTable一定已经创建了，所以该方法主要则是进行**复制**工作 
⑤需要把数据插入到链表或者树中，使用**synchronized锁定头结点**，如果这个节点是一个链表节点，那么就遍历这个链表，如果发现有相同的key值就更新value值，如果遍历完了都没有发现相同的key值，就需要在链表的尾部插入该数据。插入结束之后判断该链表节点个数是否**大于等于8**，如果大于就需要把**链表转化为红黑树存储。** 
⑥如果这个节点是一个红黑树节点，那就需要按照树的插入规则进行插入。 

⑦put结束之后，需要给map已存储的数量+1，在**addCount方法中判断是否需要扩容**

**GET方法**

Get方法不论是在HashMap和ConcurrentHashMap都是最容易理解的，它的主要步骤是： 
①先判断数组的桶中的第一个节点是否寻找的对象是为链表还是红黑树， 
②如果是红黑树另外做处理 
③如果是链表就先判断头节点是否为要查找的节点，如果不是那么就遍历这个链表查询 

④如果都不是，那就返回null值。

## ThreadLocal

http://cmsblogs.com/?p=2442

ThreadLocal定义了四个方法：

- get()：返回此线程局部变量的当前线程副本中的值。
- initialValue()：返回此线程局部变量的当前线程的“初始值”。
- remove()：移除此线程局部变量当前线程的值。
- set(T value)：将此线程局部变量的当前线程副本中的值设置为指定值。

除了这四个方法，ThreadLocal内部还有一个**静态内部类**ThreadLocalMap，该内部类才是实现线程隔离机制的**关键**，get()、set()、remove()都是基于该内部类操作。ThreadLocalMap提供了一种用键值对方式存储每一个线程的变量副本的方法，key为**当前ThreadLocal对象**，value则是对应线程的变量副本。

对于ThreadLocal需要注意的有两点：

1. ThreadLocal实例本身是**不存储值**，它只是提供了一个在当前线程中找到副本值的key。
2. 是ThreadLocal包含在Thread中，而不是Thread包含在ThreadLocal中，有些小伙伴会弄错他们的关系。

`ThreadLocal`的实现是这样的：每个`Thread` 维护一个 `ThreadLocalMap` 映射表（Thread源码中定义了一个ThreadLocalMap），这个映射表的 `key` 是 `ThreadLocal` 实例本身，`value` 是真正需要存储的 `Object`。

也就是说 `ThreadLocal` **本身并不存储值**，它只是作为一个 `key` 来让线程从 `ThreadLocalMap` 获取 `value`。值得注意的是图中的虚线，表示 `ThreadLocalMap` 是使用 `ThreadLocal` 的**弱引用**作为 `Key` 的，弱引用的对象在 GC 时会被回收。

### 内存泄漏

http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/

`ThreadLocalMap`使用`ThreadLocal`的弱引用作为`key`，如果一个`ThreadLocal`没有外部强引用来引用它，那么系统 GC 的时候，这个`ThreadLocal`势必会被回收，这样一来，`ThreadLocalMap`中就会出现`key`为`null`的`Entry`，就没有办法访问这些`key`为`null`的`Entry`的`value`，如果当前线程再迟迟不结束的话，这些`key`为`null`的`Entry`的`value`就会一直存在一条**强引用链**：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value`永远无法回收，造成内存泄漏。

其实，`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。

但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用`static`的`ThreadLocal`，延长了`ThreadLocal`的生命周期，可能导致的内存泄漏（参考[ThreadLocal 内存泄露的实例分析](http://blog.xiaohansong.com/2016/08/09/ThreadLocal-leak-analyze/)）。
- 分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就会导致内存泄漏。

### get()

> 返回当前线程所对应的线程变量

```java
    public T get() {
        // 获取当前线程
        Thread t = Thread.currentThread();

        // 获取当前线程的成员变量 threadLocal
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            // 从当前线程的ThreadLocalMap获取相对应的Entry
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")

                // 获取目标值        
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

	
	private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
	//定义为final，表示ThreadLocal一旦创建其散列值就已经确定
	private final int threadLocalHashCode = nextHashCode();
	//从0开始，每次nextHashCode都要加一次0x61c88647
	private static AtomicInteger nextHashCode = new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        //采用CAS保证线程安全
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

```

首先通过当前线程获取**所对应的成员变量ThreadLocalMap**，然后通过ThreadLocalMap获取当前ThreadLocal的Entry，最后通过所获取的Entry获取目标值result。

### set(T value)

> 设置当前线程的线程局部变量的值。

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

获取当前线程所对应的ThreadLocalMap，如果不为空，则调用ThreadLocalMap的set()方法，key就是当前ThreadLocal，如果不存在，则调用createMap()方法新建一个，如下：

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

### 总结

- ThreadLocal **不是用于解决共享变量**的问题的，也不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制。这点至关重要。
- 每个Thread内部都有一个ThreadLocal.ThreadLocalMap类型的成员变量，该成员变量用来存储**实际的ThreadLocal变量副本**。
- ThreadLocal并不是为线程保存对象的副本，它仅仅只起到一个**索引**的作用。它的主要目的是为每一个线程隔离一个类的实例，这个实例的作用范围仅限于线程内部。

## 线程池

http://cmsblogs.com/?p=2448

核心为ThreadPoolExecutor类，其父类AbstractExecutorService在submit方法中调用了子类ThreadPoolExecutor中实现的execute()方法采用了模板模式，即在抽象父类中调用了子类实现的方法

![çº¿ç¨æ± ](http://upload-images.jianshu.io/upload_images/2251324-f27c0413b40951c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 五种状态

Running, SHUTDOWN, STOP, TIDYING（整理）, TERMINATED。线程池状态通过一个**AtomicInteger的ctl值**实现，高三位为线程池状态，低29位为线程池的任务数量

**RUNNING**：处于RUNNING状态的线程池能够接受新任务，以及对新添加的任务进行处理。

**SHUTDOWN**：处于SHUTDOWN状态的线程池不可以接受新任务，但是可以对已添加的任务进行处理。

**STOP**：处于STOP状态的线程池不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。

**TIDYING**：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。

**TERMINATED**：线程池彻底终止的状态。

**线程池的线程管理机制：**首先创建核心线程，一个线程处理一个任务，若核心线程不够处理任务，则放入阻塞队列，若阻塞队列也满了，则判断是否小于设定的maximumPoolSize，即最大值，若小于，则继续创建线程，否则就执行拒绝策略。当线程数大于**corePoolSize**时，则keepAliveTime会生效，若空闲太多，则会根据设定的存活时间后自动销毁。



![1542458483689](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542458483689.png)

![1542459653512](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542459653512.png)

#### 线程池参数

**corePoolSize**

线程池中核心线程的数量。当提交一个任务时，线程池会新建一个线程来执行任务，直到当前线程数等于corePoolSize。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。

**maximumPoolSize**

线程池中允许的最大线程数。线程池的阻塞队列满了之后，如果还有任务提交，如果当前的线程数小于maximumPoolSize，则会新建线程来执行任务。注意，如果使用的是无界队列，该参数也就没有什么效果了。

**keepAliveTime**

线程空闲的时间。线程的创建和销毁是需要代价的。线程执行完任务后不会立即销毁，而是继续存活一段时间：keepAliveTime。默认情况下，该参数**只有在线程数大于corePoolSize时才会生效**。

**unit**

keepAliveTime的单位。TimeUnit

**workQueue**

用来保存等待执行的任务的阻塞队列，等待的任务必须实现Runnable接口。我们可以选择如下几种：

- ArrayBlockingQueue：基于数组结构的有界阻塞队列，FIFO。[【死磕Java并发】—-J.U.C之阻塞队列：ArrayBlockingQueue](http://cmsblogs.com/?p=2381)
- LinkedBlockingQueue：基于链表结构的有界阻塞队列，FIFO。
- SynchronousQueue：不存储元素的阻塞队列，每个插入操作都必须等待一个移出操作，反之亦然。[【死磕Java并发】—-J.U.C之阻塞队列：SynchronousQueue](http://cmsblogs.com/?p=2418)
- PriorityBlockingQueue：具有优先界别的阻塞队列。[【死磕Java并发】—-J.U.C之阻塞队列：PriorityBlockingQueue](http://cmsblogs.com/?p=2407)

**threadFactory**

用于设置**创建线程的工厂**。该对象可以通过Executors.defaultThreadFactory()。

ThreadFactory的左右就是提供创建线程的功能的线程工厂。他是通过newThread()方法提供创建线程的功能，newThread()方法创建的线程都是“非守护线程”而且“线程优先级都是Thread.NORM_PRIORITY”。

**handler**

RejectedExecutionHandler，**线程池的拒绝策略**。所谓拒绝策略，是指将任务添加到线程池中时，线程池拒绝该任务所采取的相应策略。当向线程池中提交任务时，如果此时线程池中的线程已经饱和了，而且阻塞队列也已经满了，则线程池会选择一种拒绝策略来处理该任务。

线程池提供了四种拒绝策略：

1. AbortPolicy：直接抛出异常，默认策略；
2. CallerRunsPolicy：用调用者所在的线程来执行任务；
3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
4. DiscardPolicy：直接丢弃任务；
   当然我们也可以实现自己的拒绝策略，例如记录日志等等，实现RejectedExecutionHandler接口即可。

![1542457859148](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542457859148.png)

#### 线程池类别

**Executor**框架提供了三种线程池，他们都可以通过工具类Executors来创建。

**FixedThreadPool**

FixedThreadPool，可重用固定线程数的线程池，其定义如下：

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

corePoolSize 和 maximumPoolSize都设置为创建FixedThreadPool时指定的参数nThreads，意味着当线程池满时且阻塞队列也已经满时，如果继续提交任务，则会直接走拒绝策略，该线程池不会再新建线程来执行任务，而是直接走拒绝策略。FixedThreadPool使用的是默认的拒绝策略，即AbortPolicy，则直接抛出异常。

keepAliveTime设置为0L，表示空闲的线程会立刻终止。

workQueue则是使用**LinkedBlockingQueue**，但是没有设置范围，那么则是最大值（**Integer.MAX_VALUE**），这基本就相当于一个无界队列了。使用该“无界队列”则会带来哪些影响呢？当线程池中的线程数量等于corePoolSize 时，如果继续提交任务，该任务会被添加到阻塞队列workQueue中，当阻塞队列也满了之后，则线程池会新建线程执行任务直到maximumPoolSize。由于FixedThreadPool使用的是“无界队列”LinkedBlockingQueue，那么maximumPoolSize参数无效，同时指定的拒绝策略AbortPolicy也将无效。而且该线程池也不会拒绝提交的任务，如果客户端提交任务的速度快于任务的执行，那么keepAliveTime也是一个无效参数。

**SingleThreadExecutor**

SingleThreadExecutor是使用单个worker线程的Executor，定义如下：

```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

作为单一worker线程的线程池，SingleThreadExecutor把corePool和maximumPoolSize均被设置为1，和FixedThreadPool一样使用的是无界队列LinkedBlockingQueue,所以带来的影响和FixedThreadPool一样。

**CachedThreadPool**

CachedThreadPool是一个会根据需要创建新线程的线程池 ，他定义如下：

```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

CachedThreadPool的corePool为0，maximumPoolSize为Integer.MAX_VALUE，这就意味着所有的任务一提交就会加入到阻塞队列中。keepAliveTime这是为60L，unit设置为TimeUnit.SECONDS，意味着空闲线程等待新任务的最长时间为60秒，空闲线程超过60秒后将会被终止。阻塞队列采用的SynchronousQueue，而我们在[【死磕Java并发】—-J.U.C之阻塞队列：SynchronousQueue](http://cmsblogs.com/?p=2418)中了解到SynchronousQueue是一个没有元素的阻塞队列，加上corePool = 0 ，maximumPoolSize = Integer.MAX_VALUE，这样就会存在一个问题，如果主线程提交任务的速度远远大于CachedThreadPool的处理速度，则CachedThreadPool会不断地创建新线程来执行任务，这样有可能会导致系统耗尽CPU和内存资源，所以在**使用该线程池是，一定要注意控制并发的任务数，否则创建大量的线程可能导致严重的性能问题**。







![1542459714758](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542459714758.png)

![1542460358087](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542460358087.png)

![1542460487444](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542460487444.png)

![1542460580105](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542460580105.png)

![1542460748426](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\1542460748426.png)

#### Runnable和Callable的区别

1. 两者最大的不同点是：实现Callable接口的任务线程能返回执行结果；而实现Runnable接口的任务线程不能返回结果；
2. Callable接口的call()方法允许抛出异常；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛；

#### java锁的分类图

![img](file:///C:\Users\ASUS\AppData\Roaming\Tencent\Users\739868197\TIM\WinTemp\RichOle\9TWAC)7A4~S7E35Z@8E)SNP.png)









