```java
public class LockTest {
    private static ReentrantLock lock = new ReentrantLock();

    private static ReentrantLock lock1 = new ReentrantLock(true);

    private static ReentrantReadWriteLock lock2 = new ReentrantReadWriteLock();

    private static int value = 0;
    private static Lock readLock = lock2.readLock();
    private static Lock writeLock = lock2.writeLock();

    public static int i = 0;

    public static void increase() {
        try {
            lock.lock();
            i++;
        } finally {
            lock.unlock();
        }
    }

    public static Object handleRead(Lock lock) throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);// 模拟读操作
            System.out.println("读操作:" + value);
            return value;
        } finally {
            lock.unlock();
        }
    }

    public static void handleWrite(Lock lock, int index)
            throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);// 模拟写操作
            System.out.println("写操作:" + value);
            value = index;
        } finally {
            lock.unlock();
        }
    }


    public static void main(String[] args) throws InterruptedException {
//        Thread t1 = new TestThread();
//        Thread t2 = new TestThread();
//        t1.start();
//        t2.start();
//        t1.join();
//        t2.join();
//        System.out.println(i);
//        Thread t1 = new TestThread1();
//        Thread t2 = new TestThread1();
//        t1.start();
//        t2.start();

        TestReadThread testReadThread = new TestReadThread();
        TestWriteThread testWriteThread = new TestWriteThread();
        for (int i = 0; i < 18; i++) {
            new Thread(testReadThread).start();
        }
        for (int i = 18; i < 36; i++) {
            new Thread(testWriteThread).start();
        }
    }

    static class TestThread extends Thread {

        @Override
        public void run() {
            super.run();
            for (int i = 0; i < 100; i ++) {
                LockTest.increase();
            }
        }
    }

    static class TestThread1 extends Thread {
        @Override
        public void run() {
            super.run();
            while (true) {
                try {
                    lock1.lock();
                    System.out.println(Thread.currentThread().getName() + "获得锁");
                } finally {
                    lock1.unlock();
                }
            }
        }
    }

    static class TestReadThread extends Thread {
        @Override
        public void run() {
            super.run();
            try {
                handleRead(readLock);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

    static class TestWriteThread extends Thread {
        @Override
        public void run() {
            super.run();
            try {
                handleWrite(writeLock, new Random().nextInt(100));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}

```



## 原理(JDK1.7)：

ReentrantLock#lock实际是调用内部absract类Sync#lock, Sync继承自AQS，有FairSync（公平锁）和NonFairSync（非公平锁）的两种实现

FairSync

```java
/**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is
         * implemented in subclasses, but both need nonfair
         * try for trylock method.
         非公平锁的尝试lock， tryAcquire各自的子类去实现，但是需要为两个子类的tryLock方法提供非公平
         的tryAcquire方法
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
              // 没有锁，CAS抢占锁，成功后设置锁的OwnerThread=currentThread
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
              // 如果锁已经是当前线程占有了，更新重入次数
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes this lock instance from a stream.
         * @param s the stream
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
/////////////////////////////////////////////////////////////////////////////////////////////////

/**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 公平锁，需要判断当前的队列里是否有等待锁的其他线程
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
    }

/////////////////////////////////////////////////////////////////////////////////////////////////
/**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
          // 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
         //  抢占锁失败后会调用父类（AbstractQueuedSynchronizer）的acquire取锁。 aquire里又会调用tryAcquire，最终调用非公平锁的nonfairTryAcquire(1)
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
/////////////////////////////////////////////////////////////////////////////////////////////
/**
AbstractQueuedSynchronizer
**/
class AbstractQueuedSynchronizer {
  
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
  
   public final boolean hasQueuedPredecessors() {
        Thread first = null; Node h, s;
        if ((h = head) != null && ((s = h.next) == null ||
                                   (first = s.waiter) == null ||
                                   s.prev == null))
            first = getFirstQueuedThread(); // retry via getFirstQueuedThread
        return first != null && first != Thread.currentThread();
    }

  
}
```



总结：



#### lock过程

公平锁：

lock()  -> FairSync.lock() -> acquire(1) -> tryAcquire(1) -> state==0(无锁) && 无前驱节点（在当前线程没有其他线程在wait锁）-> CAS抢占锁 -> setOwner=currentThread -> 成功获取锁

lock()  -> FairSync.lock() -> acquire(1) -> tryAcquire(1) -> state !=0(有锁) || 有前驱节点（在当前线程还有其他线程在wait锁）-> AbstractQueuedSynchronizer.acquireQueued



非公平锁：

lock() -> NonFairSync.lock() -> CAS抢占锁成功 -> setOwner=currentThread -> 成功获取锁

lock() -> NonFariSync.lock() -> CAS抢占锁失败 -> acquire(1) -> tryAcuire(1) -> nonfairTryAcquire(1)[非公平抢占锁，CAS抢占] -> 抢占失败 -> AbstractQueuedSynchronizer.acquireQueued 



#### unlock过程

公平锁和非公平锁都一样

Unlock -> Sync.release() -> AbstractQueuedSynchronizer.release() -> tryRelease()[state=0才能认为是真正的解锁成功，走unpark唤醒被挂起的线程]



非公平锁和公平锁区别：非公平锁在lock的时候立即会用CAS尝试抢占锁， 抢占成功就获得锁， 失败后进入AQS acquire锁的流程，会再次判断锁的状态，进行CAS抢占； 公平锁在lock的时候会判断有没有前驱节点，如果有的话不会去获取锁。他们如果获取不到锁最后都是通过AbstractQueuedSynchronizer.acquireQueued里的LockSupport.park, lock当前的线程

