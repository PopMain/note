### Looper



循环去取MessageQueue里不断取消息，每个线程只有一个Looper，一个Looper有一个消息队列MessageQueue，主线程：Looper.getMainLooper()可以获取，其他线程Looper.myLooper() 获取前必须使用Looper.prepare()

Looper.prepare()

```java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
  // 设置当前线程的Looper
        sThreadLocal.set(new Looper(quitAllowed));
}
```



prepare后要调用Looper.loop()启动Looper循环

```java
public static void loop() {
  			// 获取Looper
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
  			// 获取Looper的消息队列
        final MessageQueue queue = me.mQueue;

        ..........
				
        // 循环取消息
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

           ···········
        }
    }
```



### Handler

handler有一个Looper成员, 一个MessageQueue：

```java
final Looper mLooper;
final MessageQueue mQueue;
```

可以是Looper.myLooper()或者是通过构造函数初始化， 其中mQueue=mLooper.mQueue，极消息队列是mLooper中的消息队列

我们sendMsg其实是把消息加到mQueue中





### HandlerThread

继承于Thread

```Java
    @Override
    public void run() {
        mTid = Process.myTid();
        // 初始化Looper
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            // Looper初始化了，那么唤起其他线程(主要是getLooper()),获取Looper
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
      	// 启动Looper
        Looper.loop();
        mTid = -1;
    }

     public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        // 如果mLooper == null，那么等待，直到Thread.run()跑到，初始化了Looper
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }
    
```



使用

```java
HandlerThread ht = new HandlerThread();
// 启动线程
ht.start();

······

Handler h = new Handler(ht.getLooper())；
h.sendEmptyMsg()
```





