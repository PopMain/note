1. AppBarLayout设置layout_behavior="@string/fix_appbar_layout_behavior"  (实现AppBarLayout.Behavior)

2. Header的layout (AppBarLayout的child)需设置layout_scrollFlags=“scroll” (冠以scrolFlags: https://blog.csdn.net/LosingCarryJie/article/details/78917423  

    https://blog.csdn.net/eyishion/article/details/80282204)

3. 如果布局不复杂，那么可以直接使用RecyclerView作为CoordinatorLayout的NestedScrollingChildren， 对recycelerview设置layout_behavior="@string/appbar_scrolling_view_behavior" 

4. 若布局比较复杂，那么可以在RecyclerView外嵌套一层androidx.core.widget.NestedScrollView作为CoordinatorLayout的NestedScrollingChildren，对NestedScrollView设置layout_behavior="@string/appbar_scrolling_view_behavior"

5. 都需要对RecyclerView设置android:nestedScrollingEnabled="false"



关于CoordinatorLayout Behavior:

http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0224/3991.html

https://www.jianshu.com/p/b987fad8fcb4

https://juejin.im/entry/5948bd9ca0bb9f006bf062df

http://androidwing.net/index.php/70





