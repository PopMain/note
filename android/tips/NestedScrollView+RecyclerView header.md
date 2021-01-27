```xml
<androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintLeft_toLeftOf="parent" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:focusableInTouchMode="true">


           // header部分
            <TextView
                android:id="@+id/tv"
                android:layout_width="wrap_content"
                android:layout_height="100dp"
                android:textColor="#000000"
                android:textSize="24dp"
                android:textStyle="bold"
                android:text="文字文字文字文字文字"
                />

      
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recyclerView"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />

        </LinearLayout>
        
     </androidx.core.widget.NestedScrollView>
```

初始化进来是自动滑动到了recycler view部分，断点发现是focuse的时候触发了scroll, 可以在LinearLayout这里加

android:focusableInTouchMode="true"