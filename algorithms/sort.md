#### 希尔排序

```java
 public static void shellSort(int[] input) {
     int gap = input.length / 2;
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

