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
        val intercept = onTouchEvent(event)
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
```