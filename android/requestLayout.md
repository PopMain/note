### View.requestLayout





View.requestLayout ->  mParent.requestLayout -> ViewRootImpl.requestLayout



ViewRootImpl.requestLayout

```java
@Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }
// 注册帧回调
void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }

   final TraversalRunnable mTraversalRunnable = new TraversalRunnable();

   final class TraversalRunnable implements Runnable {
          @Override
          public void run() {
              doTraversal();
          }
      }
  
   void doTraversal() {
          if (mTraversalScheduled) {
              mTraversalScheduled = false;
              mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

              if (mProfile) {
                  Debug.startMethodTracing("ViewAncestor");
              }

              performTraversals();
              ...
          }
      }
      
      // 帧回调遍历中发现需要ruqeustLayout调用performLayout
      private void performTraversals() {
          ...
          boolean layoutRequested = mLayoutRequested && (!mStopped || mReportNextDraw);

          ...
          final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
          ...

          if (didLayout) {
            performLayout(lp, mWidth, mHeight);
          }
        
      }
      
     // performLayout调用host.performLayout，层层调用子view的layout触发重新布局
 			private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
            int desiredWindowHeight) {
      		
        	...
          host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
          ...
      }
```



