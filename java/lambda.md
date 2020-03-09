### Java lambda



#### 语法

```java
(parameters) -> expression

(parameters) ->{ statements; }
```



- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。



```java
public class Test {

    public static void main(String[] args) {
         // 可选类型声明 ：（a，b）不用类型声明
        fun0((a, b) -> {
            System.out.println("" + a + b);
        });
        // 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
        fun1(a -> {
                System.out.println("fun1 call");
        });
        // 可选的大括号：如果主体包含了一个语句，就不需要使用大括号
        fun0((a, b) -> System.out.println("" + a + b));
        // 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了			 一个数值
        C c = () -> false;
        // 编译错误
        // E e = () -> true;
        // 编译错误，接口只能有一个方法
        // D d = () -> false;
    }

    interface A {
        void fun();
    }

    interface B {
        void fun(int a, int b);
    }
    
     interface C {
        boolean fun();
    }
    
     interface D {
        boolean fun1();
        boolean fun2();
    }

    abstract class E {
        abstract void fun();
    }

    static void fun1(A a) {

    }

    static void fun0(B b) {

    }

    static void fun3(int i, A a) {

    }

    static void fun4(int i, B b) {

    }

    static void fun5(int i, B b, int x) {

    }

}

```



Android为什么能使用lambda: 

因为Android的编译插件会把我们的Lambda变成接口，**所以当我们Kotiln调用Java的代码时候是可以使用lambda的，但是Kotlin调用Kotlin代码不行的**



### Kotlin Lambda



#### 语法

```kotlin
声明：
(参数的类型，参数类型，...) -> 返回值类型
实现：
{参数1，参数2，... -> {操作参数的代码}

  1. 无参数的情况 ：
    val/var 变量名 = { 操作的代码 }

    2. 有参数的情况
    val/var 变量名 : (参数的类型，参数类型，...) -> 返回值类型 = {参数1，参数2，... -> 操作参数的代码}
```



- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选参数声明**: 当参数为0个或者1个的时候，不需要声明参数（1个参数使用it）
- 如果一个方法的最后一个参数是lambda,那么可以把大括号提到外面
- 如果一个方法的参数只有一个lambda,可以省略括号。
- Kotlin版本lambda表达式中不允许使用return关键字, kotlin用最后一行作为lambda的返回



```Kotlin
class Test {

    var block0  : ((Int, Int) -> Unit)? = null
    var block2 : (() -> Unit)? = null
    var block3 : ((Int) -> Unit)? = null
    var block4 : ((Int) -> Boolean)? = null
    fun main(): Boolean {
        // 可选类型声明：实现不需要声明参数类型，kotlin推导
        block0 = { a, b ->
            print(a+b)
        }
        // 可选参数声明: 当参数为0个或者1个的时候，不需要声明参数（1个参数使用it）
        block2 = {
            print("lambda called")
        }
        //
        block3 = {
            print(it)
        }
        // 如果一个方法的最后一个参数是lambda,那么可以把大括号提到外面（下面两个调用等价）
        fun1(4, { a, b ->
            print(a+b)
        })
        fun1(4) { a, b ->
            print(a+b)
        }
        // 如果一个方法的参数只有一个lambda,可以省略括号（下面两个调用等价）
        fun2() { a, b ->
            print(a+b)
        }
        fun2 { a, b ->
            print(a+b)
        }
        // Kotlin版本lambda表达式中不允许使用return关键字, kotlin用最后一行作为lambda的返回
        // 也可以通过匿名方法解决这个问题
//        lm3 = {
//            return it < 100
//        } 编译错误
        block4 = {
            it < 100
        }
        block4 = f
        block4 = {
            fun3(it)
        }
        return true
    }


    fun fun1(i: Int, l : (a: Int, b: Int) -> Unit) {

    }

    fun fun2(l : (a: Int, b: Int) -> Unit) {

    }

    fun fun3(i: Int): Boolean {
        return i < 100
    }

    private val f = fun(i: Int): Boolean {
       return when {
            i < 0 -> true
            i > 100 -> false
            i < 50 -> true
            else -> false
        }
    }

}
```





### Dart Lambda

未完待续。。。