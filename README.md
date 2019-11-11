# Algorithm
算法学习（java）
## 分治策略

在分治策略中，要递归的解决一个问题，在每层递归中应用如下三个步骤：
1. 分解：将问题划分为一些子问题，子问题的形式与原问题一样，只是规模更小
2. 解决：递归地求解出子问题，如果子问题的规模足够小，则停止递归，直接求解
3. 合并：将子问题的解组合成原问题的解
求解问题需要找出基线条件和递归条件。

### 最大子数组问题（如LeetCode 121. Best Time to Buy and Sell Stock）
Say you have an array for which the ith element is the price of a given stock on day i.

If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.

Note that you cannot sell a stock before you buy one.

Example 1:

Input: [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             Not 7-1 = 6, as selling price needs to be larger than buying price.

在此问题中，我们可以将其变换为最大子数组问题，即寻找一段时期，使得从第一天到最后一天的股票价格净变值最大。因此，我们不再从每日价格的角度去看待输入的数据，而是考察每日价格变化。也就是，寻找每日价格变化中“和”最大的非空连续子数组。

#### 使用分治策略的求解方法
假定我们要寻址子数组A[low...high]的最大子数组，使用分治技术意味着我们要将子数组划分为两个规模尽量相等的子数组。也就是说，找到子数组的中央位置，然后考虑求解两个子数组A[low...mid]和A[mid+1...high]。A[low...high]的任何连续子数组A[i..j]所处的位置必然是以下三种情况之一：
* 完全位于子数组A[low..mid]中，因此low<=i<=j<=mid
* 完全位于子数组A[mid+1..high]中，因此mid < i <= j <= high
* 跨越了中点，因此low <= i <= mid < j <= high
实际是，最大子数组必然是完全位于A[low..mid]中、完全位于A[mid + 1..high]中或者跨越重点的所有子数组中和最大者。我们可以递归地求解左右部分的最大子数组。因此，剩下的全部工作就是寻找跨越中点的最大子数组，然后在三种情况中选取和最大值。
##### 求跨越中点最大子数组
```
public static int findMaxCrossingSubArray(int[] prices, int low, int mid, int high) { 
    int left_sum = Integer.MIN_VALUE;
    int sum = 0;
    //求左半部的最大子数组
    for (int i = mid; i >= low; i--) {
        sum += prices[i];
        if (sum > left_sum) left_sum = sum;
    }
    int right_sum = Integer.MIN_VALUE;
    sum = 0;
    //求右半部的最大子数组
    for (int i = mid + 1; i <= high; i++) {
        sum += prices[i];
        if (sum > right_sum) right_sum = sum;
    }
    return right_sum + left_sum;
}
```
##### 求最大子数组问题的分治算法
```
public static int findMaxSubArray(int[] prices, int low, int high) {
    if (low == high) return prices[low];//基线条件，子数组只有一个子数组——它自身
    else {
        int mid = (low + high) / 2;
        int left_sum = findMaxSubArray(prices, low, mid);
        int right_sum = findMaxSubArray(prices, mid + 1, high);
        //完成合并工作
        int acoss_sum = findMaxCrossingSubArray(prices, low, mid, high);//求跨越中点的最大子数组
        if (left_sum >= right_sum && left_sum >= acoss_sum) return left_sum;//检测最大和子数组是否在左子数组中，若是则返回最大和
        else if (right_sum >= left_sum && right_sum >= acoss_sum) return right_sum;//检测最大和子数组是否在右子数组中
        else return acoss_sum;//则最大子数组必然跨越中点
    }
}
```
##### 求最大子数组的线性规划算法
```
public static Integer find(int[] array) {
    // 当前迭代元素之前的最大子数组之和(不包含当前迭代元素)
    Integer ahead = array[0];
    // 包含当前迭代元素的最大子数组
    Integer containCurent = array[0];
    /**
     * 当前最大子数组有两种种情况:
     * 情况1：不包含当前迭代元素，也就是 array[i].
     * 情况2:包含当前迭代元素,要么最大子数组就是当前迭代元素array[i];
     * 要么最大子数组在在array[0...i]中,在array[0...i]中的话必然和前一个元素相连,
     * 那么，当前元素加上前一个元素包含自身的最大子数组即为当前元素包含自身的最大子数组.
     */
    for (int i = 0; i < array.length; i++) {
        if ((containCurent + array[i]) >= array[i]) {
            containCurent = containCurent + array[i];
        } else {
            containCurent = array[i];
        }
        if (containCurent > ahead) {
            ahead = containCurent;
        }
    }
    return ahead;
}
```
### 归并排序算法（分治策略）
* 分解：分解待排序的n个元素的序列成各具n/2个元素的两个子序列
* 解决：使用归并排序递归地排序两个子序列
* 合并：合并两个已排序的子序列以产生已排序的答案 </br>
关键操作时“合并”步骤中两个已排序序列的合并。通过merge(A,p,q,r)来完成合并，其中A是一个数组，p、q和r是数组的下标，满足p <= q < r。假设子数组A[p..q]和A[q+1..r]都已排好序。它合并这两个子数组形成单一的已排好序的子数组代替当前的子数组A[p..r]。
```
public static void mergeSort(int[] A, int p, int r) {
        if (p < r) {
            int q = (p + r) / 2;
            mergeSort(A, p, q);
            mergeSort(A, q + 1, r);
            merge(A, p, q, r);
        }
    }
    public static void merge(int[] A, int p, int q, int r) {
        int[] left = new int[q - p + 1 + 1];
        int[] right = new int[r - q + 1];
        for (int i = 0; i < q - p + 1; i++) {
            left[i] = A[p + i];
        }
        for (int i = 0; i < r - q; i++) {
            right[i] = A[q + i + 1];
        }
        left[left.length - 1] = Integer.MAX_VALUE;
        right[right.length - 1] = Integer.MAX_VALUE;
        int i = 0, j = 0;
        for (int k = p; k <= r; k++) {
            if (left[i] <= right[j]) {
                A[k] = left[i++];
            } else A[k] = right[j++];
        }
    }
```
为避免在每个基本步骤必须检查是否有子数组为空，在每个子数组的末尾都放置了一张**哨兵**牌，它包含一个特殊的值，用于简化代码。当两个子数组都出现哨兵值的时候，说明所有非哨兵牌都已经被放置到输出数组中了。

