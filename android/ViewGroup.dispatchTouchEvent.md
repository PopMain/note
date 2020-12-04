```kotlin
/**
 * @Description: 事件分发流程初略流程伪代码
 * @Date 2020/11/11 4:45 PM
 * @Author wzx
 * @Email wzx@meitu.com
 */
class ViewGroup(context: Context?) : View(context) {

    var mFirstTouchTarget: TouchTarget? = null

    override fun dispatchTouchEvent(event: MotionEvent?): Boolean {
        // 先判断是否消费
        var handler = false
       // 是否拦截事件
        val intercept
        if (disAllowIntercept) {
          // requestDisallowInterceptTouchEvent(true)
          intercept = false
        } else {
          intercept = onInterceptTouchEvent(event)
        }
        var alreadyDispatchedToNewTouchTarget = false
        var nowTouchTarget: TouchTarget? = null
        if (!intercept) {
             if (event?.action ==MotionEvent.ACTION_DOWN) {
                 // 只有down才走这里
                 allChildrenViews.forEach { child ->
                     // 判断child是否在touch区域
                     if (child在touch区域) {
                         if (child.dispatchTouchEvent(event)) {
                             // 子view分发事件，如果子view消费了事件，设置touch的子view
                             nowTouchTarget = addTouchTarget(child)
                             alreadyDispatchedToNewTouchTarget = true
                         }
                     }
                 }
             }
        }
        if (mFirstTouchTarget == null) {
            // 没有子view消费了事件, 调用View.dispatchTouchEvent
            handler = super.dispatchTouchEvent(event)
        } else {
            // 有子view消费了事件, 分发子View的dispatchTouchEvent
            if (alreadyDispatchedToNewTouchTarget && nowTouchTarget == mFirstTouchTarget) {
                handler = true
            } else {
                handler = mFirstTouchTarget.dispatchTouchEvent(event)
            }
        }
        return handler
    }

    private fun addTouchTarget(
        child: View) {
        var target = bindTouchTarget(child, pointerIdBits)
        mFirstTouchTarget = target
        return target
    }

}


/**
** View.dispatchTouchEvent
**/
class View {
  
    /**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.
     *
     * @param event The motion event to be dispatched.
     * @return True if the event was handled by the view, false otherwise.
     */
    public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
           // 先走OnTouchListener
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
            // 没有再走onTouchEvent(onClickListener在onTouchEvent中调用)
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
}

```