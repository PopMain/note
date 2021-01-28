1. n数据不一致

   线程A进行put操作的时候计算了要put的位置，计算得到了链表的位置，这个时候让出了时间片，线程B获取了时间片并且对同个HashMap进行put，这个时候刚好put的位置和位置和线程A的位置一样， 线程Bput后线程A继续进行put, 这个时候就会覆盖Bput的值

2. 循环链表

   引起： 扩容resize

   扩容核心：JDK 1.7

   ```java
    void resize(int newCapacity) {
           Entry[] oldTable = table;
           int oldCapacity = oldTable.length;
           if (oldCapacity == MAXIMUM_CAPACITY) {
               threshold = Integer.MAX_VALUE;
               return;
           }
   
           Entry[] newTable = new Entry[newCapacity];
           transfer(newTable);
           table = newTable;
           threshold = (int)(newCapacity * loadFactor);
       }
   
   void transfer(Entry[] newTable, boolean rehash) {  
           int newCapacity = newTable.length;  
           for (Entry<K,V> e : table) {  
  
               while(null != e) {  
                Entry<K,V> next = e.next;           
                   if (rehash) {  
                    e.hash = null == e.key ? 0 : hash(e.key);  
                   }  
                int i = indexFor(e.hash, newCapacity);   
                   e.next = newTable[i];  
                newTable[i] = e;  
                   e = next;  
            } 
           }  
    }  
   
static int indexFor(int h, int length) {
           return h & (length-1);
    }
   ```

   这个方法的功能是将原来的记录重新计算在新桶的位置，然后迁移过去。

   假如现在有HashMap: [3, A]>[7, B]> [5, C]，他们在同一个链表上, 线程Thread1和线程2同时要进行put，并且触发了resize

   Thread1-> transfer: e=[A, 3], next=[7,B], 此时系统调度让出时间分片，Thread2进行resize, reszie完的结果是

   0：

   1：[5, C] -> null

   2:

   3: [7, B] -> [3, A] -> null

   Thread2让出时间片，Thread1继续刚才的代码，**此时e=[3, A], next=[7, B]**

   indexFor(e.hash, newCapacity)=3

   e.next=newTable[3]=null (**注意，线程1线程2中的newTable宿主是独立的，是在方法中new的**)

   newTable[3]=e=[3, A];

   e=next=[7, B];

   此时结构：

   0：

   1： 

   2：

   3：[3, A] -> null
   
   线程1进入第二次循环， **e=[7, B], next=[3, A]**（Thread1进行resize后[7, B]->[3, A]）
   
   e.next=newTable[3]=[3, A];
   
   newTable[3]=e=[7,B];
   
   e=next=[3,A];
   
   此时结构：
   
   0：
   
   1： 
   
   2：
   
   3：[7, B] -> [3, A] -> null

​        线程1进入第三次循环， **e=[3, A], next=null**

​      e.next=newTable[3]=[7,B]

​     此时循环链表出现 [7,B]<=>[3, A]