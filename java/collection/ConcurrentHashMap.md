### ConcurrentHashMap

#### JDK 1.7

Segement分段锁，每个Segment维护一定数量的桶，多线程可以同时访问不同Segment的桶，从而提高并发（并发度就是Segment的个数，默认是16）

Segment继承自ReentrantLock.



**put:**

```java
public V put(K key, V value) {
        Segment<K,V> s;
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        int j = (hash >>> segmentShift) & segmentMask;
        // 通过hash定位到segment(原子操作)
        // 如果对应的segment为空那么通过CAS+自旋锁创建新的segment
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }

// Segment put
// 首先第一步的时候会尝试获取锁tryLock，如果获取失败肯定就有其他线程存在竞争，则利用scanAndLockForPut() 自旋获取锁。
// Segment中put的大体流程是先对Segment加锁，然后根据(tab.length-1)&hash找到对应的slot，然后遍历slot对应的链表，如果key对应的entry已经存在，根据onlyIfAbsent标志决定是否替换旧值，如果key对应的entry不存在，创建新节点插入链表头部，期间若容量超过限制，判断是否需要进行rehash
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
            HashEntry<K,V> node = tryLock() ? null :
                scanAndLockForPut(key, hash, value);
            V oldValue;
            try {
                HashEntry<K,V>[] tab = table;
                int index = (tab.length - 1) & hash;
                HashEntry<K,V> first = entryAt(tab, index);
                for (HashEntry<K,V> e = first;;) {
                    if (e != null) {
                        K k;
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        e = e.next;
                    }
                    else {
                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        int c = count + 1;
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
                unlock();
            }
            return oldValue;
        }

// 自旋尝试次数
static final int MAX_SCAN_RETRIES =
            Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

/**
查找是否有要put的hash的HashEntry是否已经存在， 如果不存在创建一个, 并且如果对应的链表不为空，会遍历，找到key相等的节点，或者到链表尾部开始自旋次数倒计时
在开始自旋倒计时后会没隔一次进行判断，看first是否变了（有插入或者移除），如果有的话重新遍历。
**/
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
            HashEntry<K,V> first = entryForHash(this, hash);
            HashEntry<K,V> e = first;
            HashEntry<K,V> node = null;
            int retries = -1; // negative while locating node
    		 // 尝试获取锁，如果获取不到，那么就是有竞争，while自旋
            while (!tryLock()) {
                HashEntry<K,V> f; // to recheck first below
                if (retries < 0) {
                    if (e == null) {
                        // e节点为空
                        if (node == null) // speculatively create node
                            // node为空，创建HashEntry
                            node = new HashEntry<K,V>(hash, key, value, null);
                        // retries = 0 进入 ++retries分支，开始自旋次数计数
                        retries = 0;
                    }
                    else if (key.equals(e.key))
                        // 如果链表中当前节点e的key和目标的key equal的话说明找到了，retries = 0 进入 ++retries分支开始自旋倒计时
                        retries = 0;
                    else
                        // e指向链表的下一个node
                        e = e.next;
                }
                else if (++retries > MAX_SCAN_RETRIES) {
                    // 如果自旋次数大于MAX_SCAN_RETRIES阻塞锁获取
                    lock();
                    break;
                }
                else if ((retries & 1) == 0 &&
                         (f = entryForHash(this, hash)) != first) {
                   // (retries & 1) == 0每隔一次自旋循环判断首节点有是否有变动，若有，更新first，重置retries，停止自旋计数，重新扫描
                    e = first = f; // re-traverse if entry changed
                    retries = -1;
                }
            }
            return node;
        }

 private void rehash(HashEntry<K,V> node) {
            HashEntry<K,V>[] oldTable = table;
            int oldCapacity = oldTable.length;
            // 扩大两倍
            int newCapacity = oldCapacity << 1;
            threshold = (int)(newCapacity * loadFactor);
            HashEntry<K,V>[] newTable =
                (HashEntry<K,V>[]) new HashEntry[newCapacity];
            int sizeMask = newCapacity - 1;
            for (int i = 0; i < oldCapacity ; i++) {
                HashEntry<K,V> e = oldTable[i];
                if (e != null) {
                    HashEntry<K,V> next = e.next;
                    int idx = e.hash & sizeMask;
                    if (next == null)   
                        // 桶内只有一个Node e, 直接把这个节点放到新的桶 e.has & (newCapacity - 1)
                        newTable[idx] = e;
                    else { 
                       // 如果桶内不止有一个Node
                        HashEntry<K,V> lastRun = e;
                        int lastIdx = idx;
                        for (HashEntry<K,V> last = next;
                             last != null;
                             last = last.next) {
                            int k = last.hash & sizeMask;
                            if (k != lastIdx) {
                                lastIdx = k;
                                lastRun = last;
                            }
                        }
                        newTable[lastIdx] = lastRun;
                        // Clone remaining nodes
                        //复制lastRun之前的所有节点
                        for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                            V v = p.value;
                            int h = p.hash;
                            int k = h & sizeMask;
                            HashEntry<K,V> n = newTable[k];
                            newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                        }
                    }
                }
            }
            int nodeIndex = node.hash & sizeMask; // add the new node
            node.setNext(newTable[nodeIndex]);
            newTable[nodeIndex] = node;
            table = newTable;
        }

```

**key -> hash -> 定位到哪个Segment -> 尝试获取锁 -> 如果获取不到进入自旋，自旋过程遍历HashEntry并且创建节点，自旋次数到达后直接阻塞获取锁-> 加锁后扫描链表put** 



**get:**

只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。

由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。

ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**。



**size:**

每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。

在执行 size 操作时，需要遍历所有 Segment 然后把 count 累计起来。

ConcurrentHashMap 在执行 size 操作时先尝试不加锁，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的。

尝试次数使用 RETRIES_BEFORE_LOCK 定义，该值为 2，retries 初始值为 -1，因此尝试次数为 3。

如果尝试的次数超过 3 次，就需要对每个 Segment 加锁。

```java
/**
 * Number of unsynchronized retries in size and containsValue
 * methods before resorting to locking. This is used to avoid
 * unbounded retries if tables undergo continuous modification
 * which would make it impossible to obtain an accurate result.
 */
static final int RETRIES_BEFORE_LOCK = 2;

public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            // 超过尝试次数，则对每个 Segment 加锁
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            // 连续两次得到的结果一致，则认为这个结果是正确的
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```

containsKey也是用类似的方法， 三次查找，每次查找累加每个Segment的modCount(修改次数)， 如果三次的值都不一致，那么就所有段都加锁

#### clear

在无锁下执行的，这带来的后果就是数据的不一致: 考虑这么一种情况，当A线程执行clear方法时，已经将segments[0]对象清空了，此时B线程执行put(key,value)方法，如果key散列到segments[0]上，那么A执行完后容器中还有元素！**所以clear方法是弱一致的**



1、段内加锁的：put，putIfAbsent，remove，replace、clear等

2、不加锁的：get，containsKey等

3、整个数据结构加锁的：size，containsValue、isEmpty等



#### JDK 1.8

抛弃了原有的Segment分段锁，采用了CAS + synchronized同步锁保证并发安全性



**put:** 

```java
 final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                // tab为空，初始化tab
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                 // tabAt原子操作获取对应的桶（Node）, 如果为空，原子操作 + 自旋（for循环内）创建一个新的node
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                // 如果fh == -1, 那么需要扩容
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // 否则对应的桶加锁
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```





**get:**

```java
Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
int h = spread(key.hashCode());
if ((tab = table) != null && (n = tab.length) > 0 &&
    // 原子读
    (e = tabAt(tab, (n - 1) & h)) != null) {
    if ((eh = e.hash) == h) {
        if ((ek = e.key) == key || (ek != null && key.equals(ek)))
            return e.val;
    }
    else if (eh < 0)
        return (p = e.find(h, key)) != null ? p.val : null;
    while ((e = e.next) != null) {
        if (e.hash == h &&
            ((ek = e.key) == key || (ek != null && key.equals(ek))))
            return e.val;
    }
}
return null;
```

根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
如果是红黑树那就按照树的方式获取值。
就不满足那就按照链表的方式遍历获取值。











参考： https://zhuanlan.zhihu.com/p/40327960

https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/classes/java/util/concurrent/ConcurrentHashMap.java

https://github.com/frohoff/jdk8u-jdk/blob/master/src/share/classes/java/util/concurrent/ConcurrentHashMap.java



