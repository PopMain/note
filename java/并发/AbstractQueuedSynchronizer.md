## AQS

核心代码：

```java
public class AbstractQueuedSynchronizer {
  
  protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
  
  public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
  
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
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
  
  
   public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
  
  private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
  
  private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        return true;
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

  
   public final boolean hasQueuedPredecessors() {
        Thread first = null; Node h, s;
        if ((h = head) != null && ((s = h.next) == null ||
                                   (first = s.waiter) == null ||
                                   s.prev == null))
            first = getFirstQueuedThread(); // retry via getFirstQueuedThread
        return first != null && first != Thread.currentThread();
    }

  
   private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
}
```



维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列)

AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。

- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。

- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。

- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。

- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

  以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

  再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

  　　一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。

### 加锁

获取锁的时候需要先tryAcquire, tryAcquire是需要子类去实现的方法，用来判断当前是否能去获取锁（可以实现公平、非公平锁）

tryAcquire失败后进入acquireQueued流程:

1. 获取当前节点的前驱节点
2. 判断前驱节点是否为头节点，这里有两种情况：
   - 前驱节点为头节点，说明当前节点前面没有节点在等待获取锁资源，只需要等待前驱节点释放锁资源。所以可以尝试抢锁，这里有两种情况：
     - 抢锁成功，则将当前节点设置为头节点，将当前节点前驱节点的后置引用设置为空，返回false
     - 抢锁失败，说明头节点还没有释放锁资源，此时将当前节点挂起。这里有两种情况：
       - 如果挂起成功，则线程等待被唤醒，唤醒之后再次判断前驱节点是否为头节点。
       - 如果挂起失败，再次判断前驱节点是否为头节点。
   - 前驱节点不是头节点，说明当前节点前面有其他节点在等待获取锁资源，此时将当前节点挂起。
3. 如果在挂起阶段发生异常，则取消抢锁。
4. 这里为无限循环，直到线程获取到锁资源或者取消抢锁才会退出循环。



获取锁失败后会进入shouldParkAfterFailedAcquire判断当前线程是否需要挂起（LockSupport.park）:

首先获取当前节点的前驱节点的状态，这里有三种情况：
\* 前驱节点的状态为SIGNAL。其中，SIGNAL表明该节点在释放锁资源后应该将后置节点唤醒。返回true。
\* 前驱节点的状态为CANCELLED。CANCELLED表明该节点已取消抢锁，此时将从当前节点开始向前寻找，直到找到一个节点的状态不为CANCELLED，然后将他设置为当前节点的前驱节点。之后返回false.
\* 如果前驱节点的状态不是以上两种情况，则通过CAS将前驱节点的状态设置为SIGNAL，之后返回false。



### 解锁

  tryRelease尝试释放锁资源，这里有两种情况：

- 成功释放锁资源，获取到AQS链表中头节点，判断头节点是否为空，这里有两种情况：
  - 如果头节点为空，说明没有节点持有锁资源，返回true.
  - 如果头节点不为空，判断头节点状态是否为0：
    - 如果头节点状态为0，说明阻塞队列中没有线程在等待获取锁，返回true.
    - 如果头节点状态不为0，则将阻塞队列中第一个等待获取锁资源的线程唤醒。随后返回ture.



unparkSuccessor唤醒后置节点，当持有锁的节点执行相关代码完成后，需要释放锁资源并唤醒后置节点。

1. 首先获取头节点的状态，如果小于0则通过CAS将状态设置为0.
2. 获取头节点的后置节点，这里有两种情况：
   - 如果头节点的后置节点为空或者头节点的后置节点的状态大于0，则将头节点的后置节点置为空，同时从AQS链表的尾节点向前搜索，直到找到最后一个节点状态小于等于0的节点，将该节点唤醒。
   - 如果头节点的后置节点不为空，则直接将该节点唤醒。

**用unpark()唤醒等待队列中最前边的那个未放弃线程**，这里我们也用s来表示吧。此时，再和acquireQueued()联系起来，s被唤醒后，进入if (p == head && tryAcquire(arg))的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire()寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire()的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire()也返回了！！And then, DO what you WANT!



参考： https://www.cnblogs.com/waterystone/p/4920797.html