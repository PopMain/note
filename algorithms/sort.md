#### 希尔排序

时间复杂度：

不断细化分组（减小分组内数与数之间的步长，直到步长等于1，即分组数等于数组长度），每次都都对分组内的数进行插入排序，這樣可以讓一個元素可以一次性地朝最終位置前進一大步。然後算法再取越來越小的步長進行排序，算法的最後一步就是普通的[插入排序](https://zh.m.wikipedia.org/wiki/插入排序)，但是到了這步，需排序的數據幾乎是已排好的了（此時[插入排序](https://zh.m.wikipedia.org/wiki/插入排序)較快）

```java
 public static void shellSort(int[] input) {
<<<<<<< Updated upstream
     int gap = input.length / 2;
=======
        // 步长
        int gap = input.length / 2;
>>>>>>> Stashed changes
        while (gap > 0) {
            System.out.println("gap="+gap);
            for (int i = 0; i < gap; i ++) {
                for (int realIndex = i; realIndex < input.length; realIndex += gap) {
                    for (int j = realIndex; j > i; j = j - gap) {
                        if (input[j] < input[j - gap]) {
                            Utils.swap(input, j, j - gap);
                        }
                    }
                }
            }
            gap = gap / 2;
        }
    }
```

