### 饿汉

```java
class Singleton {
    static instance = new Singleton()
}
```

```kotlin
object Singleton
//实际二进制bytecode
public final class SingletonDemo {
   public static final SingletonDemo INSTANCE;
   private SingletonDemo(){}
   static {
      SingletonDemo var0 = new SingletonDemo();
      INSTANCE = var0;
   }
}
```

### 懒汉

```java
public class SingletonDemo {
    private static SingletonDemo instance;
    private SingletonDemo(){}
    public static SingletonDemo getInstance(){
        if(instance==null){
            instance=new SingletonDemo();
        }
        return instance;
    }
}

```

```kotlin
class SingletonDemo private constructor() {
    companion object {
        private var instance: SingletonDemo? = null
            get() {
                if (field == null) {
                    field = SingletonDemo()
                }
                return field
            }
        fun get(): SingletonDemo{
        //细心的小伙伴肯定发现了，这里不用getInstance作为为方法名，是因为在伴生对象声明时，内部已有getInstance方法，所以只能取其他名字
         return instance!!
        }
    }
}
```

### 线程安全式懒汉

```java
public class SingletonDemo {
    private static SingletonDemo instance;
    private SingletonDemo(){}
    public static synchronized SingletonDemo getInstance(){//使用同步锁
        if(instance==null){
            instance=new SingletonDemo();
        }
        return instance;
    }
}
```



```kotlin
class SingletonDemo private constructor() {
    companion object {
        private var instance: SingletonDemo? = null
            get() {
                if (field == null) {
                    field = SingletonDemo()
                }
                return field
            }
        @Synchronized
        fun get(): SingletonDemo{
            return instance!!
        }
    }

}
```



### 双重校验锁式

```java
public class SingletonDemo {
    private volatile static SingletonDemo instance;
    private SingletonDemo(){} 
    public static SingletonDemo getInstance(){
        if(instance==null){
            synchronized (SingletonDemo.class){
                if(instance==null){
                    instance=new SingletonDemo();
                }
            }
        }
        return instance;
    }
}
```



```kotlin
class SingletonDemo private constructor() {
    companion object {
        val instance: SingletonDemo by lazy(mode = LazyThreadSafetyMode.SYNCHRONIZED) {
        SingletonDemo() }
    }
}

// lazy内部实现
public fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
        when (mode) {
            LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
            LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
            LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
        }

// Lazy接口
public interface Lazy<out T> {
     //当前实例化对象，一旦实例化后，该对象不会再改变
    public val value: T
    //返回true表示，已经延迟实例化过了，false 表示，没有被实例化，
    //一旦方法返回true，该方法会一直返回true,且不会再继续实例化
    public fun isInitialized(): Boolean
}
// SynchronizedLazyImpl
private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            //判断是否已经初始化过，如果初始化过直接返回，不在调用高级函数内部逻辑
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                }
                else {
                    val typedValue = initializer!!()//调用高级函数获取其返回值
                    _value = typedValue   //将返回值赋值给_value,用于下次判断时，直接返回高级函数的返回值
                    initializer = null
                    typedValue  
                }
            }
        }
		//省略部分代码
}
/**
委托属性语法是： val/var <属性名>: <类型> by <表达式>。在 by 后面的表达式是该 委托， 因为属性对应的 get()（和 set()）会被委托给它的 getValue() 和 setValue() 方法。 属性的委托不必实现任何的接口，但是需要提供一个 getValue() 函数（和 setValue()——对于 var 属性）
**/
```



### 静态内部类式

```java
//Java实现
public class SingletonDemo {
    private static class SingletonHolder{
        private static SingletonDemo instance=new SingletonDemo();
    }
    private SingletonDemo(){
        System.out.println("Singleton has loaded");
    }
    public static SingletonDemo getInstance(){
        return SingletonHolder.instance;
    }
}
```

```kotlin
//kotlin实现
class SingletonDemo private constructor() {
    companion object {
        val instance = SingletonHolder.holder
    }

    private object SingletonHolder {
        val holder= SingletonDemo()
    }

}
```





### 带参数

```kotlin
class SingletonDemo private constructor(private val property: Int) {
    companion object {
        @Volatile private var instance: SingletonDemo? = null
        fun getInstance(property: Int) =
                instance ?: synchronized(this) {
                    instance ?: SingletonDemo(property).also { instance = it }
                }
    }
}
```