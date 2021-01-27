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
       	// 判断线程是否启动，未启动直接返回null
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
// getLooper()要在start()后调用，否则会返回null
Handler h = new Handler(ht.getLooper())；
h.sendEmptyMsg()
```







### Looper中的死循环为什么不会卡住？会吃掉CPU的资源吗？

答案：当然不会

why?

```java
public static void main(String[] args) {
        ....

        //创建Looper和MessageQueue对象，用于处理主线程的消息
        Looper.prepareMainLooper();

        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread(); 

        //建立Binder通道 (创建新线程)
        thread.attach(false);

        Looper.loop(); //消息循环运行
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

/* thread.attach(false)；便会创建一个Binder线程（具体是指ApplicationThread，Binder的服务端，用于接收系统服务AMS发送来的事件），该Binder线程通过Handler将Message发送给主线程 */

class MessageQuene {
    boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            // 这里唤醒 nativePollOnce 的沉睡
            nativeWake(mPtr);
        }
    }
    return true;
}
    
    Message next() {
    //...
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        // nativePollOnce 这里陷入沉睡, 等待唤醒
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }
            //...
        }
        //...
    }
}
}

```



进入loop千会先thread.attach(false)，创建了ApplicaitonThread，实现了Binder, 用于接收系统的事件

 取消息队列的时候调用了nativePollOnce， 最终是调用了
 int eventCount = epoll_wait(mEpollFd, eventItems, EPOLL_MAX_EVENTS, timeoutMillis);

检测文件描述符（管道）事件，当消息队列为空，且任何消息到达时候此时主线程会释放CPU资源进入休眠状态

enqueueMessage添加消息的时候会调用nativeWake，最终调用

ssize_t nWrite = TEMP_FAILURE_RETRY(write(mWakeEventFd, &inc, sizeof(uint64_t)));

往同一个管道里写，唤起休眠的主线程



### epoll_wait

 int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
等待事件的产生，类似于select()调用。参数events用来从内核得到事件的集合，maxevents告之内核这个events有多大，这个 maxevents的值不能大于创建epoll_create()时的size，参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。如果返回–1，则表示出现错误，需要检查 errno错误码判断错误类型。

第1个参数 epfd是 epoll的描述符。

第2个参数 events则是分配好的 epoll_event结构体数组，epoll将会把发生的事件复制到 events数组中（events不可以是空指针，内核只负责把数据复制到这个 events数组中，不会去帮助我们在用户态中分配内存。内核这种做法效率很高）。

第3个参数 maxevents表示本次可以返回的最大事件数目，通常 maxevents参数与预分配的events数组的大小是相等的。

第4个参数 timeout表示在没有检测到事件发生时最多等待的时间（单位为毫秒），如果 timeout为0，则表示 epoll_wait在 rdllist链表中为空，立刻返回，不会等待。



关于ET、LT两种工作模式：

epoll有两种工作模式：LT（水平触发）模式和ET（边缘触发）模式。

默认情况下，epoll采用 LT模式工作，这时可以处理阻塞和非阻塞套接字，而上表中的 EPOLLET表示可以将一个事件改为 ET模式。ET模式的效率要比 LT模式高，它只支持非阻塞套接字。

 ET模式与LT模式的区别在于：

当一个新的事件到来时，ET模式下当然可以从 epoll_wait调用中获取到这个事件，可是如果这次没有把这个事件对应的套接字缓冲区处理完，在这个套接字没有新的事件再次到来时，在 ET模式下是无法再次从 epoll_wait调用中获取这个事件的；而 LT模式则相反，只要一个事件对应的套接字缓冲区还有数据，就总能从 epoll_wait中获取这个事件。因此，在 LT模式下开发基于 epoll的应用要简单一些，不太容易出错，而在 ET模式下事件发生时，如果没有彻底地将缓冲区数据处理完，则会导致缓冲区中的用户请求得不到响应。默认情况下，Nginx是通过 ET模式使用 epoll的。