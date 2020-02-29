1. ```java
   Object.wait()方法可以暂停线程，并释放对象锁
   Object.notify()方法可以唤醒需要该对象锁的其他线程，并在执行完后续步骤，到了synchronized临界区后，才会把锁释放
   ```

2.

```
wait()方法暂停线程执行，并立即释放对象锁

notify()/notifyAll() 方法唤醒其他等待该对象锁的线程,并在执行完同步代码块中的后续步骤后，释放对象锁

notify()和notifyAll()的区别在于：
    notify只会唤醒其中一个线程，
    notifyAll则会唤醒全部线程。
    
至于notify会唤醒哪个线程，是由线程调度器决定的。
```



3. ```java
   wait()是暂停当前线程。
   
   notify()则是唤醒等待当前对象锁的线程
   ```



4 . wait&notify实现生产消费者模型

```java
package com.company;

import java.util.PriorityQueue;

public class Main4 {

    private static int SIZE = 10;
    private static final PriorityQueue<Integer> queue = new PriorityQueue<>(SIZE);

    public static void main(String[] args) {
        Consumer consumer = new Consumer();
        Producer producer = new Producer();
        new Thread(consumer).start();
        new Thread(producer).start();

    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            while (true) {
//                System.out.println("Consumer run");
                synchronized (queue) {
                    while (queue.size() == 0) {
                        try {
                            System.out.println("队列为空，等待成员进入");
                            // 队列空，消费者现场挂起，释放queue的锁，生产者获得锁后开始生产
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            System.out.println(e.toString());
                            queue.notify();
                        }
                    }
                    queue.poll();
                    queue.notify();
                    System.out.println("向队列取中取出一个元素，队列大小：" + queue.size());
                }
            }
        }
    }

    static class Producer implements Runnable {
        @Override
        public void run() {
            while (true) {
//                System.out.println("Producer run");
                synchronized (queue) {
                    while (queue.size() == SIZE) {
                        try {
                            System.out.println("队列已满，等待剩余空间");
                             // 队列满，生产者者现场挂起，释放queue的锁，消费获得锁后开始消费
                            queue.wait();
                        } catch (InterruptedException e) {
                            System.out.println(e.toString());
                            queue.notify();
                        }
                    }
                    queue.offer(1);
                    queue.notify();
                    System.out.println("向队列中插入一个元素，队列大小：" + queue.size());
                }
            }
        }
    }
}

```

