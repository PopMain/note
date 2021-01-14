### View.post流程



先看源码：

```java
public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }
```

如果mAttachInfo不为空，使用 mAttachInfo.mHandler.post。否则getRunQueue().post



#### mAttachInfo

Q: 哪来的？ 

A: View.dispatchAttachedToWindow

Q： View.dispatchAttachedToWindow哪里被调用

A：父布局ViewGroup.dispatchAttachedToWindow里会调用super.dispatchAttachedToWindow

所以这个mAttachInfo是跟着布局一层一层通过dispatchAttachedToWindow， 最终我们需要看最外层的View(Activity的DecorView)的dispatchAttachedToWindow是怎么被调用的。 

ViewRootImpl中dispatchAttachedToWindow的调用流程（DecorView与ViewRootImpl关系ViewRootImpl.md）：



scheduleTraversals() -> **Choregrapher.postCallback(TranversalRunnalbe)** -> doTranversal() -> performTraversals() -> dispatchAttachedToWindow()



其中performTraversals调用dispatchAttachedToWindow有一个mFirst标记的判断

```java
void performTraversals() {
	...
  if (mFirst) {
    ...
    host.dispatchAttachedToWindow(mAttachInfo, 0);
    ...
  }
  ...
  mFirst = false;
}
```

也就是只有首次performTraversals的时候才会调用dispatchAttachedToWindow



(Choregrapher详见Choregrapher.md)





#### HandlerActionQueue

如果为mAttachInfo为空，则通过getRunQueue获取HandlerActionQueue实例调用HandlerActionQueue.post

啥是HandlerActionQueue？

```java
public class HandlerActionQueue {
    private HandlerAction[] mActions;
    private int mCount;
	
  public void post(Runnable action) {
        postDelayed(action, 0);
    }

    public void postDelayed(Runnable action, long delayMillis) {
        final HandlerAction handlerAction = new HandlerAction(action, delayMillis);

        synchronized (this) {
            if (mActions == null) {
                mActions = new HandlerAction[4];
            }
            mActions = GrowingArrayUtils.append(mActions, mCount, handlerAction);
            mCount++;
        }
    }
  
    public void executeActions(Handler handler) {
          synchronized (this) {
              final HandlerAction[] actions = mActions;
              for (int i = 0, count = mCount; i < count; i++) {
                  final HandlerAction handlerAction = actions[i];
                  handler.postDelayed(handlerAction.action, handlerAction.delay);
              }

              mActions = null;
              mCount = 0;
          }
      }
}
```

一个HandlerAction的数组， HandlerAction是对Runnable的一次封装。 HandlerActionQueue.post实际上是对Runnable进行一次封装

放到一个数组里。 那么这个数组在HandlerActionQueue.executeActions中读取，HandlerActionQueue.executeActions什么时候被调用呢？还是在View.dispatchAttachedToWindow中。



```java
void dispatchAttachedToWindow(AttachInfo info, int visibility) {
  ...
    
    if (mRunQueue != null) {
        mRunQueue.executeActions(info.mHandler);
        mRunQueue = null;
    }
  ...
}
```

