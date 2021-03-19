### Hanlder#postDelayed



postDelayed -> sendMessageDelayed

最终调用：

```java
 public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

 public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

SystemClock.uptimeMillis获取系统从开机启动到现在的时间，期间不包括休眠的时间，是一个相对的时间

```java
class Looper {
  public static void loop() {
    、、、
       for (;;) {
            Message msg = queue.next(); 
         、、、
       }
    、、、
  }
}

class MessageQueue {
   Message next() {
      for (;;) {
        、、、
          // 挂起
        nativePollOnce(ptr, nextPollTimeoutMillis);
        、、、
          final long now = SystemClock.uptimeMillis();
       、、、
        if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.下一条消息尚未准备好。设置一个超时，以便在准备就绪时唤醒
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
                }
        、、、
      }
   }
}
```

next()中如果当前链表头部消息是延迟消息，则根据延迟时间进行消息队列会阻塞，不返回给Looper message，知道时间到了，返回给message

如果在阻塞中有新的消息插入到链表头部则唤醒线程

