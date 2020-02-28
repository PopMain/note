



1. HashMap：containKey,containValue（key或者value）, 根据key.hashCode获取node， node==key或者key.equals(node), 所以必须实现hashCode和equals方法

2. HashMap添加一个Node<k,v>

   node[(n-1)&hash(k)]=v

   hash(k) = (h = key.hashCode()) ^ (h >>> 16)

3. HashSet实际是HashMap拿key来保存, vlue保存一个obj：

   ```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
   ```

   

4. HashMap、HashTable区别

   HashTable是同步的所以是线程安全的，HashMap不是；HashMap允许有null的key、value，HashTable不允许；HashMap有containKey和containValue而HashTable只有contain方法; HashTable直接使用key的hashCode,HashMap则重新计算（2的hash()方法）

5. HashMap以hash值为标识分散到hash桶里（key的hash值一致的连表）

6. Object#hashCode():内存地址的哈希码

7. 扩容：当表中的75%已经被占用，即视为需要扩容了,扩容两倍

8. Android中使用SparseArray，ArrayMap代替HashMap

9. Equals()方法原则：

   自反性。x.equals(x)要等于true

   对称性。x.equals(y) = true, 那么 y.equals(x) = ture

   传递性。x.equals(y) = ture, y.equals(z) = ture, 那么 x.equals(z) = true

   一致性。若x,y的比较信息没有变，不管执行几次x.equals(y)都返回true

   对任何不是null的x, x.equals(null)都返回false

10. hashCode()：result = 37 * result + c ：c为某个filed的hashCode

   