

# 查找和排序

## 注意事项

1. 查找常用顺序查找、二分查找、哈希表查找和二叉排序树查找。
2. 无论使用循环还是递归，面试官都期待候选人**可以信手拈来写出二分查找法的代码**。
3. 算法题要求在**排序的数组（或者部分排序的数组）**中**查找一个数字**或者**统计某个数字出现的频次**，都可以尝试使用二分查找法来进行解题。
4. 一定要对各种排序算法的特点烂熟于胸，可以从额外空间消耗、平均时间复杂度、最差时间复杂度等方面比较其优缺点。

## 11. 旋转数组的最小数字

[https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

> 把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  
>
> **示例 1：**
>
> ```java
> 输入：[3,4,5,1,2]
> 输出：1
> ```
>
> **示例 2：**
>
> ```java
> 输入：[2,2,2,0,1]
> 输出：0
> ```

### 思路

旋转数组，分左右两段有序序列，可考虑使用二分查找法解题。

1. 二分查找法，左右指针找中间值。**右边的有序序列 >= 左边的有序序列**。

2. **中间值 >= 左指针的值**，证明中间值在左边序列，令**左指针 = 中间值**；**中间值 <= 右指针的值**，证明中间值在右边序列，令**右指针 = 中间值**。

3. 当**左右指针相邻**时，左指针指向左侧有序序列的最后一个元素，右指针指向右侧序列的第一个元素，此时说明**右指针的元素为最小值**。

4. 特殊情况1：**旋转数组旋转0次，数组只有一个递增序列**。中间值初始赋给左指针指向的元素，如果左指针指向的元素 < 右侧指针的元素，那么直接返回最小值即可。

5. 特殊情况2：**左指针的值 = 右指针的值 = 中间值**，则利用二分查找法继续找最小值，需要利用遍历法找最小值。

   > 举例：[0,1,1,1,1] 旋转为 [1,0,1,1,1] 和 [1,1,1,0,1]  无法确定索引下标2的元素到底属于左边序列，还是右边序列。

### 代码实现

源码参看：[https://github.com/duktig666/algorithm/blob/main/src/offer/MinNumberInRotatedArray11.java](https://github.com/duktig666/algorithm/blob/main/src/offer/MinNumberInRotatedArray11.java)

```java
public class MinNumberInRotatedArray11 {

    /**
     * 查找旋转数组中的最小值
     */
    public int minArray(int[] numbers) {
        // 左右两个指针
        int index1 = 0, index2 = numbers.length - 1;
        // 初始中间值指向index1，数组只有一个升序序列，直接返回第一个元素（即旋转0个元素的情况）
        int mid = index1;
        while (numbers[index1] >= numbers[index2]) {
            // 左右指针相邻，index2右侧指针代表最小值
            if (index2 - index1 == 1) {
                mid = index2;
                break;
            }

            mid = index1 + ((index2 - index1) >> 1);

            // index1 = index2 = mid 无法用二分法进行判断，需要遍历找出最小值
            if (numbers[index1] == numbers[index2] && numbers[mid] == numbers[index1]) {
                return minInOrder(numbers, index1, index2);
            }
            //二分法指针调整
            if (numbers[mid] >= numbers[index1]) {
                index1 = mid;
            } else if (numbers[mid] <= numbers[index2]) {
                index2 = mid;
            }
        }
        return numbers[mid];
    }

    /**
     * 遍历法 找出指定序列的最小值
     */
    private int minInOrder(int[] numbers, int index1, int index2) {
        int min = numbers[index1];
        for (int i = index1 + 1; i < index2; i++) {
            if (min > numbers[i]) {
                min = numbers[i];
            }
        }
        return min;
    }

}
```

