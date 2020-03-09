

https://juejin.im/post/5b3054c66fb9a00e4d53ef75

### Thread#join()

**将几个并行线程的线程合并为一个单线程执行**，应用场景是 **当一个线程必须等待另一个线程执行完毕才能执行时**



```
public class MyThread extends Thread {

    @Override
    public void run() {
        try {
    		   Thread.sleep(5000);
    		} catch(Exception e) {
    		   
    		}
        System.out.println("子线程执行完毕");
    }
}

```



``` java
public class TestThread {

    public static void main(String[] args) {
        //循环五次
        for (int i = 0; i < 5; i++) {

            MyThread thread = new MyThread();
            //启动线程
            thread.start();
            try {
                //调用join()方法
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("主线程执行完毕");
            System.out.println("~~~~~~~~~~~~~~~");

        }
    }
}


```

输出：

~~~~~~~~~~~~~~~
子线程执行完毕
主线程执行完毕

```
子线程执行完毕
主线程执行完毕
```

子线程执行完毕
主线程执行完毕

```
子线程执行完毕
主线程执行完毕
```

子线程执行完毕
主线程执行完毕
~~~~~~~~~~~~~~~



join源码：

```java
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }

```



当Thread#isAlive()那么当前线程wait并且释放锁





```java
/** Notifies the group that the thread {@code t} has terminated.
  * 通知线程组，t线程已经终止。
  */
void threadTerminated(Thread t) {
        synchronized (this) {
            remove(t);                          //从线程组中删除此线程

            if (nthreads == 0) {                //当线程组中线程数为0时
                notifyAll();                    //唤醒所有待定中的线程
            }
            if (daemon && (nthreads == 0) &&
                (nUnstartedThreads == 0) && (ngroups == 0))
            {
                destroy();
            }
        }
    }


```



当线程终止时会调用threadTerminated，notifyAll()唤醒需要该对象锁的其他线程



**Join(long maxWaitTime)**:

当前线程最多等待maxWaitTime毫秒时长，如果maxWaitTime毫秒后目标线程还没有执行完的话不再等待。其本质是调用wait(0)和wait(maxWaitTime)

