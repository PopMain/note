1. 数组在初始化的时候要指定类型

   ```java
   class Holder<T> {
       T[] arry;
       Holder(int size) {
           // array = new T[size];
           // 上写法不能通过编译, JVM通过
           // anewarray：创建引用类型数组
           // newarray int/float创建基础类型数组
           // 两个命令都要指定数组类型
           array = (T[]) new Object[size];
       }
       
   }
   ```

   

2. 类型擦除&java.lang.ArrayStoreException

   java泛型编译后都会转成Object，在传递赋值的时候进行类型转换；

   Java数组为了保证类型安全，在初始化的时候就指定了类型；

    ```java
Object[] ducks = new Integer[10];// 模拟擦除
ducks[0] = "Hello";//java.lang.ArrayStoreException
    ```

 

```java
 class Holder {
 	private T[] tArray;

    public void set(T[] ts) {
        tArray = ts;
    }

    public T[] get() {
        return tArray;
    }
 
 }

class A {
    
}

class B extends A {
    
}

class C extends A {
    
}

void main() {
    Holder<A> holder = new Holder();
    A[] as = new A[10];
    holder.set(as);
    holder.s(new B()); // ArrayStoreException
}
 
```



Object[]数组可以是任何数组的父类，或者说，任何一个数组都可以向上转型成它在定义时指定元素类型的父类的数组，这个时候如果我们往里面放不同于原始数据类型 但是满足后来使用的父类类型的话，编译不会有问题，但是在运行时会检查加入数组的对象的类型，于是会抛*ArrayStoreException*：

