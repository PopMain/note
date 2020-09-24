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

