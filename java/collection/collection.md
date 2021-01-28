



1. HashMap:

   结构大概是一个数据组根据hash值存放，每个节点有是一个链表存key,hash值一样的称之为hash桶

   Tab[(n-1)&hash(k)] [(n-1)&hash(k) ] [(n-1)&hash(k)] [(n-1)&hash(k) ] [(n-1)&hash(k) ]

   ​             |

   ​            [k1]

   ​			 |

   ​			[K2]

2. HashMap：containKey,containValue（key或者value）, 根据key.hashCode获取node， node==key或者key.equals(node), 所以必须实现hashCode和equals方法

3. HashMap添加一个Node<k,v>

   node[(n-1)&hash(k)]=v

   hash(k) = (h = key.hashCode()) ^ (h >>> 16)

4. HashSet实际是HashMap拿key来保存, vlue保存一个obj：

   ```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
   ```

   

5. HashMap、HashTable区别

   HashTable是同步的所以是线程安全的，HashMap不是；HashMap允许有null的key、value，HashTable不允许；HashMap有containKey和containValue而HashTable只有contain方法; HashTable直接使用key的hashCode,HashMap则重新计算（2的hash()方法）

6. HashMap以hash值为标识分散到hash桶里（key的hash值一致的连表）

7. Object#hashCode():内存地址的哈希码

8. 扩容：当表中的75%已经被占用，即视为需要扩容了,扩容两倍

9. Android中使用SparseArray，ArrayMap代替HashMap

10. Equals()方法原则：

   自反性。x.equals(x)要等于true

   对称性。x.equals(y) = true, 那么 y.equals(x) = ture

   传递性。x.equals(y) = ture, y.equals(z) = ture, 那么 x.equals(z) = true

   一致性。若x,y的比较信息没有变，不管执行几次x.equals(y)都返回true

   对任何不是null的x, x.equals(null)都返回false

11. hashCode()：result = 37 * result + c ：c为某个filed的hashCode

    

12. 关于红黑树：

    ```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                       boolean evict) {
            Node<K,V>[] tab; Node<K,V> p; int n, i;
            if ((tab = table) == null || (n = tab.length) == 0)
              // table是空,那么resize
                n = (tab = resize()).length;
            if ((p = tab[i = (n - 1) & hash]) == null)
              //  (n - 1) & hash 下标位置是空的，那么直接newNode,放进(n - 1) & hash位置
                tab[i] = newNode(hash, key, value, null);
            else {
                Node<K,V> e; K k;
                if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                  // (n - 1) & hash 下标位置不是是空的，但是(n - 1) & hash 下标位置的key是一样的
                    e = p;
                // 以下的部分表示已经发生了hash碰撞了
                else if (p instanceof TreeNode)
                    // 当前的结构已经是红黑树了，直接put一个树节点
                    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
                else {
                    //
                    for (int binCount = 0; ; ++binCount) {
                        if ((e = p.next) == null) {
                            p.next = newNode(hash, key, value, null);
                            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              // 当hash桶的个数大于等于8的时候要进行树形化
                                treeifyBin(tab, hash);
                            break;
                        }
                        if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                            break;
                        p = e;
                    }
                }
                if (e != null) { // existing mapping for key
                    V oldValue = e.value;
                    if (!onlyIfAbsent || oldValue == null)
                        e.value = value;
                    afterNodeAccess(e);
                    return oldValue;
                }
            }
            ++modCount;
            if (++size > threshold)
                resize();
            afterNodeInsertion(evict);
            return null;
        }
    
    
    //将桶内所有的 链表节点 替换成 红黑树节点
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果当前哈希表为空，或者哈希表中元素的个数小于 进行树形化的阈值(默认为 64)，就去新建/扩容
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            // 果哈希表中的元素个数超过了 树形化阈值，进行树形化
            // e 是哈希表中指定位置桶里的链表节点，从第一个开始
            TreeNode<K,V> hd = null, tl = null; //红黑树的头、尾节点
            do {
                //新建一个树形节点，内容和当前链表节点 e 一致
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null) //确定树头节点
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);  
            //让桶的第一个元素指向新建的红黑树头结点，以后这个桶里的元素就是红黑树而不是链表了
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
    
      TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
      }
    ```

    

   13. ```java
       
            public static void main(String[] args) throws IOException {
              System.out.println(tableSizeFor(10));
           }
       
           static final int MAXIMUM_CAPACITY = 1 << 30;
       
       /**
            * Returns a power of two size for the given target capacity.
            返回大于cap值最近的2的整数次幂的数， 比如10，则返回16
            */
           static final int tableSizeFor(int cap) {
               int n = cap - 1;
               System.out.println("=================start============");
               System.out.println(Integer.toBinaryString(n));
               System.out.println("n>>>1 = " + Integer.toBinaryString(n>>>1));
               n |= n >>> 1;
               System.out.println("n |= n >>> 1 = " + Integer.toBinaryString(n));
               System.out.println("n>>>2 = " + Integer.toBinaryString(n>>>2));
               n |= n >>> 2;
               System.out.println("n |= n >>> 2 = " + Integer.toBinaryString(n));
               System.out.println("n>>>4 = " + Integer.toBinaryString(n>>>4));
               n |= n >>> 4;
               System.out.println("n |= n >>> 4 = " + Integer.toBinaryString(n));
               System.out.println("n>>>8 = " + Integer.toBinaryString(n>>>8));
               n |= n >>> 8;
               System.out.println("n |= n >>> 8 = " + Integer.toBinaryString(n));
               System.out.println("n>>>16 = " + Integer.toBinaryString(n>>>16));
               n |= n >>> 16;
               System.out.println("n |= n >>> 16 = " + Integer.toBinaryString(n));
               return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
           }
       ```

        输出：=================start============
1001
n>>>1 = 100
n |= n >>> 1 = 1101
n>>>2 = 11
n |= n >>> 2 = 1111
n>>>4 = 0
n |= n >>> 4 = 1111
n>>>8 = 0
n |= n >>> 8 = 1111
n>>>16 = 0
n |= n >>> 16 = 1111
16





14. ```java
    /**
    扩容：
    **/
    final Node<K,V>[] resize() {
     2     Node<K,V>[] oldTab = table;
     3     int oldCap = (oldTab == null) ? 0 : oldTab.length;
     4     int oldThr = threshold;
     5     int newCap, newThr = 0;
     6     if (oldCap > 0) {
     7         // 超过最大值就不再扩充了，就只好随你碰撞去吧
     8         if (oldCap >= MAXIMUM_CAPACITY) {
     9             threshold = Integer.MAX_VALUE;
    10             return oldTab;
    11         }
    12         // 没超过最大值，就扩充为原来的2倍
    13         else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
    14                  oldCap >= DEFAULT_INITIAL_CAPACITY)
    15             newThr = oldThr << 1; // double threshold
    16     }
    17     else if (oldThr > 0) // initial capacity was placed in threshold
    18         newCap = oldThr;
    19     else {               // zero initial threshold signifies using defaults
    20         newCap = DEFAULT_INITIAL_CAPACITY;
    21         newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    22     }
    23     // 计算新的resize上限
    24     if (newThr == 0) {
    25 
    26         float ft = (float)newCap * loadFactor;
    27         newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
    28                   (int)ft : Integer.MAX_VALUE);
    29     }
    30     threshold = newThr;
    31     @SuppressWarnings({"rawtypes"，"unchecked"})
    32         Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    33     table = newTab;
    34     if (oldTab != null) {
    35         // 把每个bucket都移动到新的buckets中
    36         for (int j = 0; j < oldCap; ++j) {
    37             Node<K,V> e;
    38             if ((e = oldTab[j]) != null) {
    39                 oldTab[j] = null;
    40                 if (e.next == null)
    41                     newTab[e.hash & (newCap - 1)] = e;
    42                 else if (e instanceof TreeNode)
    43                     ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
    44                 else { // 链表优化重hash的代码块
    45                     Node<K,V> loHead = null, loTail = null;
    46                     Node<K,V> hiHead = null, hiTail = null;
    47                     Node<K,V> next;
    48                     do {
    49                         next = e.next;
    50                         // 原索引
    51                         if ((e.hash & oldCap) == 0) {
    52                             if (loTail == null)
    53                                 loHead = e;
    54                             else
    55                                 loTail.next = e;
    56                             loTail = e;
    57                         }
    58                         // 原索引+oldCap
    59                         else {
    60                             if (hiTail == null)
    61                                 hiHead = e;
    62                             else
    63                                 hiTail.next = e;
    64                             hiTail = e;
    65                         }
    66                     } while ((e = next) != null);
    67                     // 原索引放到bucket里
    68                     if (loTail != null) {
    69                         loTail.next = null;
    70                         newTab[j] = loHead;
    71                     }
    72                     // 原索引+oldCap放到bucket里
    73                     if (hiTail != null) {
    74                         hiTail.next = null;
    75                         newTab[j + oldCap] = hiHead;
    76                     }
    77                 }
    78             }
    79         }
    80     }
    81     return newTab;
    82 }
    ```





15

为什么是 为什么HashMap每次扩容都要2的倍数？

从数组hash桶分布说起tab[(n - 1) & hash]

当n是2的整数次幂时，n-1会生成一个0x111*的补码， n-1 & hash相当于对hash对数组长度求模，即当红n是2的 整数次幂的时候，(n - 1) & hash = hash % (n-1)

但是，对于现代的处理器来说，除法和求余数（模运算）是最慢的动作。 这个也是为什么HashMap每次扩容都要2的倍数。



16. hash(key)  = (h = key.hashCode()) ^ (h >>> 16)

为什么要右移？为什么要右移？

这段代码叫“**扰动函数**”。右位移16位，正好是32bit的一半，自己的高半区和低半区做异或，就是为了**混合原始哈希码的高位和低位，以此来加大低位的随机性**。

这段代码是为了对key的hashCode进行扰动计算，防止不同hashCode的高位不同但低位相同导致的hash冲突。简单点说，就是为了把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响。



reference： https://tech.meituan.com/2016/06/24/java-hashmap.html