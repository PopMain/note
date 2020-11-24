



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

    