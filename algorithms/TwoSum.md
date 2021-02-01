双指针

```java
public static int[] twoSum(int[] numbers, int target) {
		if  (numbers == null || numbers.length < 2) {
			return null;
		}
		int len = numbers.length;
		int left = 0;
		int right = len - 1;
		while (left < right) {
			int sum = numbers[left] + numbers[right];
			if (sum == target) {
				return new int[]{left, right};
			} else  if (sum > target) {
				right--;
			} else  {
				left++;
			}
		}
		return null;
}
```

