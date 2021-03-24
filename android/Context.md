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

