

##  Application的Context的构建过程

在[应用进程与系统进程之间的通信](https://duanqz.github.io/2016-01-29-Activity-IPC)一文中，介绍过应用进程启动时，需要和系统进程进行通信:

- 当应用进程在初始化自己的主线程ActivityThread时，便会发起跨进程调用**IActivityManager.attachApplication()**，告诉系统进程(SystemServer)：我已经在Linux的世界诞生了，现在需要增加Android的属性(应用的包信息、四大组件信息、Android进程名等)，才能成为一个真正的Android进程。
- 系统进程在进行包解析时，就获取了所有应用程序的静态信息。在AMS中执行一个应用进程的**attachApplication()**时，便会将这些信息的数据结构准备好，发起跨进程调用**IApplicationThread.bindApplication()**，传送应用进程启动所必需的信息。

经过以上的交互，应用进程就进入**ActivityThread.handleBindApplication()**，开始构建自己所需的Android环境了

##  关于View.getContext的类型
1. 系统小于5.0 且 Activity继承自AppCompatActivity,  XML里的TextView、ImageView等getContext返回的不是Activity本身而是TintContextWrapper

   原因： AppCompatActivity替换了LayoutInflaterFactory -》AppCompatDelegateImpl -》AppCompatViewInflater.onCreateView -》

   把TextView、ImageView等替换成AppCompatXXX -》AppCompatXXX构造方法会对传进来的Context做wrap: TintContextWrapper.wrap(context)

   ````java
   public AppCompatButton(Context context, AttributeSet attrs, int defStyleAttr) {
           super(TintContextWrapper.wrap(context), attrs, defStyleAttr);
     。。。
           }
   ````

   

2. Dialog的Context会被包装成ContextWrapper

   ```kotlin
   class MyDialog: DialogFragment() {
   
       override fun onCreateView(
           inflater: LayoutInflater,
           container: ViewGroup?,
           savedInstanceState: Bundle?
       ): View? {
           return super.onCreateView(inflater, container, savedInstanceState)
       }
   }
   ```

   这个inflater是哪里来的？

   DialogFragmet:

   ```java
   public Dialog onCreateDialog(Bundle savedInstanceState) {
       return new Dialog(getActivity(), getTheme());
   }
   
   @Override
   public LayoutInflater onGetLayoutInflater(Bundle savedInstanceState) {
       if (!mShowsDialog) {
           return super.onGetLayoutInflater(savedInstanceState);
       }
       mDialog = onCreateDialog(savedInstanceState);
       if (mDialog != null) {
           setupDialog(mDialog, mStyle);
           return (LayoutInflater) mDialog.getContext().getSystemService(
                   Context.LAYOUT_INFLATER_SERVICE);
       }
       return (LayoutInflater) mHost.getContext().getSystemService(
               Context.LAYOUT_INFLATER_SERVICE);
   }
   ```

​       如果mDialog != null，会使用mDialog.getContext，

```java
  Dialog(@NonNull Context context, @StyleRes int themeResId, boolean createContextThemeWrapper) {
        if (createContextThemeWrapper) {
            if (themeResId == Resources.ID_NULL) {
                final TypedValue outValue = new TypedValue();
                context.getTheme().resolveAttribute(R.attr.dialogTheme, outValue, true);
                themeResId = outValue.resourceId;
            }
          // Context被包裹了一次, 转成了ContextThemeWrapper
            mContext = new ContextThemeWrapper(context, themeResId);
        } else {
            mContext = context;
        }
```
 注意： 如果在5.0系统以下， Activity使用了AppCompatActivity， 那么Dilog里的TextView、ImageView等getContext会是TintContextWrapper类型的： ContextThemeWrapper又被包了一层，即

TintContextWrapper(ContextThemeWrapper(Activity))



Solution:

递归像上查找，知道找到Activity类型

```java
@Nullable
public static Activity getActivity(@NonNull View view) {
    if (null != view) {
        Context context = view.getContext();
        while (context instanceof ContextWrapper) {
            if (context instanceof Activity) {
                return (Activity) context;
            }
            context = ((ContextWrapper) context).getBaseContext();
        }
    }

    return null;
}
```

