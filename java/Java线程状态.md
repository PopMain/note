```java
 public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```



#### blocked与waiting

blocked： 进入同步锁（Thread state for a thread blocked waiting for a monitor lock），无法进入synchronized同步代码块。与blocked相关联的是Object对应的同步队列。

waiting：调用Object.wait()， Thread.join(), LockSupport.park进入。与waiting相关联的是Object对应的等待队列。

ThreadA ---> ObjectA.wait() ---> ThreadA线程进入等待队列 --->ThreadA进入waiting状态

ThreadB ----> ObjectA.notifyAll() --->  ThreadA进入同步队列 ----> ThreadA进入blocked状态， 此时ThreadB执行完同步块后，JVM会从同步队列中选取一个线程调度

```java
public class Test {

    static final Object lock = new Object();
    static volatile int value = 1;

    public static void main(String[] args) {
        Runnable ra = () -> {
            while (true) {
                synchronized (lock) {
                    System.out.println("Thread A Start: " + value);
                    while (value == 1) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    value = 1;
                    System.out.println("Thread A End: " + value);
                    lock.notifyAll();
                    System.out.println("Thread  A after notify all");
                }
            }
        };

        Runnable rb = () -> {
                while (true) {
                    synchronized (lock) {
                        System.out.println("Thread  B Start: " + value);
                        while (value == 0) {
                            try {
                                lock.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        value = 0;
                        System.out.println("Thread  B End: " + value);
                        lock.notifyAll();
                        System.out.println("Thread  B after notify all");
                    }
                }
        };
        Thread ta = new Thread(ra);
        Thread tb = new Thread(rb);
        ta.start();
        tb.start();
    }
}
```

#### Time Waiting

- `Thread.sleep`
- `Object.wait` with timeout
- `Thread.join` with timeout
- `LockSupport.parkNanos`
- `LockSupport.parkUntil`



#### Sleep

**sleep()方法**正在执行的线程主动让出CPU（然后CPU就可以去执行其他任务），在sleep指定时间后CPU再回到该线程继续往下执行。在sleep指定时间后CPU再回到该线程继续往下执行(注意：sleep方法只让出了CPU，而并不会释放同步资源锁！！！)；wait()方法则是指当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行，只有调用了notify()方法，之前调用wait()的线程才会解除wait状态，可以去参与竞争同步资源锁，进而得到执行。（注意：notify的作用相当于叫醒睡着的人，而并不会给他分配任务，就是说notify只是让之前调用wait的线程有权利重新参与线程的调度）；
————————————————

2、sleep()方法可以在任何地方使用；wait()方法则只能在同步方法或同步块中使用；

————————————————

3、sleep()是线程线程类（Thread）的方法，调用会暂停此线程指定的时间，但监控依然保持，不会释放对象锁，到时间自动恢复；wait()是Object的方法，调用会放弃对象锁，进入等待队列，待调用notify()/notifyAll()唤醒指定的线程或者所有线程，才会进入锁池，不再次获得对象锁才会进入运行状态；
————————————————