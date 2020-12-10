```java


private static final Unsafe unsafe = Unsafe.getUnsafe();
// value内存偏移
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
// value值，volatile修饰，保证可见性
private volatile int value;
// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
 /*
 * 其中getIntVolatile和compareAndSwapInt都是native方法
 * getIntVolatile是获取当前的期望值
 * compareAndSwapInt就是我们平时说的CAS(compare and swap)，通过比较如果内存区的值没有改变，那么就用新值直接给该内存区赋值
 */
// paramObject是当前的实例，paramLong value内存偏移（地址），parmInt变化值
public final int getAndAddInt(Object paramObject, long paramLong, int paramInt) {
  int i;
  do {
      // 根据内存偏移获取此时实际的值比如一开始读到的是5
      i = this.getIntVolatile(paramObject, paramLong);
      // 根据内存偏移量读到实际的value值和i进行比较，如果一样，说明value被更新过了，就更新值为i+paramInt,不一样在走会循环继续getIntVolatile
      // 此时获取的i已经是实际的value值了，比较成功后可以更新了
      // compareAndSwapInt通过native走系统调用，能保证原子性（保证再比较、更新过程中数据是不会被修改的）
  } while(!this.compareAndSwapInt(paramObject, paramLong, i, i + paramInt)));
  return var5;
}
/**
**
unsafe类是CAS的核心类，由于Java无法直接操作底层系统，只能通过native修饰的本地方法操作，基于unsafe类就可以直接操作内存中的数据。
getAndAddInt(Object var1, long var2, int var4)中：Object var1：当前对象，long var2：当前对象的内存地址，int var4：需要更新的值，这里就是1。
var5 = this.getIntVolatile(var1, var2);：就是取到当前对象的内存值；
假如现在存在两个线程，跑在不同的cpu上，，AtomicInteger中value原始值为5，即主内存中为5，根据JMM模型，线程A和线程B各自持有一份value为5的副本分别在各自的工作内存中，
线程A通过getIntVolatile(var1, var2)拿到value值5，这时线程A被挂起。
线程B也通过getIntVolatile(var1, var2)方法获取到value值5，线程B没有被挂起，并执行compareAndSwapInt方法比较内存值也为5，成功修改内存值为6。
这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值(5)和主内存的值(6)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功
*/

// CAS （CompareAndSwap）: 中心思想就是再对数据进行操作的时候，拿当前的值和内存中的实际的值进行比较，如果一样的话才更新，不一样的话就不更新 // // //compareAndSwap通过native走系统调用，能保证原子性（保证再比较、更新过程中数据是不会被修改的）


public class ABA {

    // 普通的原子类，存在ABA问题
    AtomicInteger a1 = new AtomicInteger(10);
    // 带有时间戳的原子类，不存在ABA问题，第二个参数就是默认时间戳，这里指定为0
    AtomicStampedReference<Integer> a2 = new AtomicStampedReference<Integer>(10, 0);

    public static void main(String[] args) {
        ABA a = new ABA();
        a.test();
    }

    public void test() {
        new Thread1().start();
        new Thread2().start();
        new Thread3().start();
        new Thread4().start();
    }

    class Thread1 extends Thread {
        @Override
        public void run() {
            a1.compareAndSet(10, 11);
            a1.compareAndSet(11, 10);
        }
    }
    class Thread2 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(200);  // 睡0.2秒，给线程1时间做ABA操作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("AtomicInteger原子操作：" + a1.compareAndSet(10, 11) + ", " + a1.get());
        }
    }
    class Thread3 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(500);  // 睡0.5秒，保证线程4先执行
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int stamp = a2.getStamp();
            a2.compareAndSet(10, 11, stamp, stamp + 1);
            stamp = a2.getStamp();
            a2.compareAndSet(11, 10, stamp, stamp + 1);
        }
    }
    class Thread4 extends Thread {
        @Override
        public void run() {
            int stamp = a2.getStamp();
            try {
                Thread.sleep(1000);  // 睡一秒，给线程3时间做ABA操作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("AtomicStampedReference原子操作:" + a2.compareAndSet(10, 11, stamp, stamp + 1) + ", " + a2
            .getReference());
        }
    }
}

```

输出：

AtomicInteger原子操作：true, 11 （更新成功了，但是实际上这个时候的10已经不是原来的10）
AtomicStampedReference原子操作:false, 10（更新失败，版本号不一样了，认为现在的10是被修改过了）



ABA的危害：

Top -> 1 -> 2 -> 3

Thread1: pop动作， 在执行compareAndSet前sleep，模拟线程切换, 此时top=1, next=2

Thread2: pop 1, 2 在 push1， 此时 Top -> 1 -> 3, 执行完后切到Thread1继续top.compareAndSet(t, next), 此时top=1=t, 认为没有被修改过，可以执行替换，top->next=2与预期的 top->3不符合。这个就是所谓ABA问题

```java
import java.util.concurrent.atomic.AtomicReference;

public class ABATest {

    static class Stack {
        // 将top放在原子类中
        private AtomicReference<Node> top = new AtomicReference<>();
        // 栈中节点信息
        static class Node {
            int value;
            Node next;

            public Node(int value) {
                this.value = value;
            }
        }
        // 出栈操作
        public Node pop() {
            for (;;) {
                // 获取栈顶节点
                Node t = top.get();
                if (t == null) {
                    return null;
                }
                // 栈顶下一个节点
                Node next = t.next;
                // CAS更新top指向其next节点
                String thread = Thread.currentThread().getName();
                if ("thread1".equals(thread)) {
                    try {
                        System.out.println("thread1 popping: before sleep : top " + t + " t.next " + (t.next == null) + " t.value " + t.value);
                        if (next != null) {
                            System.out.println("thread1 popping: before sleep : next " + next + " next.value " + next.value);
                        }
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                if (top.compareAndSet(t, next)) {
                    // 把栈顶元素弹出，应该把next清空防止外面直接操作栈
                    t.next = null;
                    return t;
                }
            }
        }
        // 入栈操作
        public void push(Node node) {
            for (;;) {
                // 获取栈顶节点
                Node next = top.get();
                // 设置栈顶节点为新节点的next节点
                node.next = next;
                // CAS更新top指向新节点
                if (top.compareAndSet(next, node)) {
                    return;
                }
            }
        }
    }

    private static void testStack() {
        // 初始化栈为 top->1->2->3
        Stack stack = new Stack();
        stack.push(new Stack.Node(3));
        stack.push(new Stack.Node(2));
        stack.push(new Stack.Node(1));

        // 线程1出栈一个元素
        Thread thread1 = new Thread(() -> {
            stack.pop();
        });
        thread1.setName("thread1");
        thread1.start();
        Thread thread2 = new Thread(()->{
            // 线程2出栈两个元素
            Stack.Node A = stack.pop();
            Stack.Node B = stack.pop();
            System.out.println("push A " + A);
            // 线程2又把A入栈了
            stack.push(A);
        });
        thread2.setName("thread2");
        thread2.start();


        try {
            Thread.sleep(5000);
            Stack.Node node;
            while ((node =stack.pop()) != null) {
                System.out.println("value " + node.value);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        testStack();
    }
}

```

