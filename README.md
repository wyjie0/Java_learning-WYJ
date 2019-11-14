# Algorithm
算法学习（java）
根据自己学习《算法导论》的顺序，每一部分附加一些LeetCode上面的题目（也会从大牛那儿借鉴内容），记录自己算法的学习过程。
## [分治策略](https://github.com/wyjie0/Algorithm/issues/1)

在分治策略中，要递归的解决一个问题，在每层递归中应用如下三个步骤：
1. 分解：将问题划分为一些子问题，子问题的形式与原问题一样，只是规模更小（从中分解还是从头开始分解）
2. 解决：递归地求解出子问题，如果子问题的规模足够小，则停止递归，直接求解（如何定义最小子问题）
3. 合并：将子问题的解组合成原问题的解（如何合并）
求解问题需要找出基线条件和递归条件。

### 最大子数组问题（如LeetCode 121. Best Time to Buy and Sell Stock）
```
Say you have an array for which the ith element is the price of a given stock on day i.

If you were only permitted to complete at most one transaction (i.e., buy one and sell one share of the stock), design an algorithm to find the maximum profit.

Note that you cannot sell a stock before you buy one.

Example 1:

Input: [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
             Not 7-1 = 6, as selling price needs to be larger than buying price.
```

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

### LeetCode 241 Different Ways to Add Parentheses（from grandyang）
先建立一个结果 res 数组，然后遍历 input 中的字符，根据上面的分析，我们希望在每个运算符的地方，将 input 分成左右两部分，从而扔到递归中去计算，从而可
以得到两个整型数组 left 和 right，分别表示作用两部分各自添加不同的括号所能得到的所有不同的值，此时我们只要分别从两个数组中取数字进行当前的运算符计
算，然后把结果存到 res 中即可。当然，若最终结果 res 中还是空的，那么只有一种情况，input 本身就是一个数字，直接转为整型存入结果 res 中即可
```
Given a string of numbers and operators, return all possible results from computing all the different possible ways to group numbers and operators. The valid operators are +, - and *.

Example 1:

Input: "2-1-1"
Output: [0, 2]
Explanation: 
((2-1)-1) = 0 
(2-(1-1)) = 2
```
```
public List<Integer> diffWaysToCompute(String input) {
    List<Integer> ret = new ArrayList<>();
    for (int i = 0; i < input.length(); i++) {
        char op = input.charAt(i);
        if (!(op >= '0' && op <= '9')) {
            List<Integer> left = diffWaysToCompute(input.substring(0, i));
            List<Integer> right = diffWaysToCompute(input.substring(i + 1));
            for (int l : left) {
                for (int r : right) {
                    if (op == '+') ret.add(l + r);
                    if (op == '-') ret.add(l - r);
                    if (op == '*') ret.add(l * r);
                }
            }
        }
    }
    if (ret.size() == 0) ret.add(Integer.valueOf(input));
    return ret;
}

//使用hashmap减少重复计算
private static Map<String, List<Integer>> map = new HashMap<>();
public static List<Integer> diffWaysToCompute(String input) {
    if (map.containsKey(input)) return map.get(input);
    List<Integer> ret = new ArrayList<>();
    for (int i = 0; i < input.length(); i++) {
        char op = input.charAt(i);
        if (!(op >= '0' && op <= '9')) {
            List<Integer> left = diffWaysToCompute(input.substring(0, i));
            List<Integer> right = diffWaysToCompute(input.substring(i + 1));
            for (int l : left) {
                for (int r : right) {
                    if (op == '+') ret.add(l + r);
                    if (op == '-') ret.add(l - r);
                    if (op == '*') ret.add(l * r);
                }
            }
        }
    }
    if (ret.size() == 0) ret.add(Integer.valueOf(input));
    map.put(input, ret);
    return ret;
}
```

### 求两个有序数组中的中位数 LeetCode 4（from grandyang）

这道题让我们求两个有序数组的中位数，而且限制了时间复杂度为 O(log (m+n))，看到这个时间复杂度，自然而然的想到了应该使用二分查找法来求解。但是这道题被
定义为 Hard 也是有其原因的，难就难在要在两个未合并的有序数组之间使用二分法，如果这道题只有一个有序数组，让求中位数的话，估计就是个 Easy 题。对于这道
题来说，可以将两个有序数组混合起来成为一个有序数组再做吗，图样图森破，这个时间复杂度限制的就是告诉你金坷垃别想啦。还是要用二分法，而且是在两个数组之间
使用，感觉很高端啊。回顾一下中位数的定义，如果某个有序数组长度是奇数，那么其中位数就是最中间那个，如果是偶数，那么就是最中间两个数字的平均值。这里对于
两个有序数组也是一样的，假设两个有序数组的长度分别为m和n，由于两个数组长度之和 m+n 的奇偶不确定，因此需要分情况来讨论，对于奇数的情况，直接找到最中间
的数即可，偶数的话需要求最中间两个数的平均值。为了简化代码，不分情况讨论，使用一个小 trick，分别找第 (m+n+1) / 2 个，和 (m+n+2) / 2 个，然后求其平
均值即可，这对奇偶数均适用。若 m+n 为奇数的话，那么其实 (m+n+1) / 2 和 (m+n+2) / 2 的值相等，相当于两个相同的数字相加再除以2，还是其本身。

好，这里需要定义一个函数来在两个有序数组中找到第K个元素，下面重点来看如何实现找到第K个元素。首先，为了避免拷贝产生新的数组从而增加时间复杂度，使用两
个变量i和j分别来标记数组 nums1 和 nums2 的起始位置。然后来处理一些 corner cases，比如当某一个数组的起始位置大于等于其数组长度时，说明其所有数字均已
经被淘汰了，相当于一个空数组了，那么实际上就变成了在另一个数组中找数字，直接就可以找出来了。还有就是如果 K=1 的话，只要比较 nums1 和 nums2 的起始位
置i和j上的数字就可以了。难点就在于一般的情况怎么处理？因为需要在两个有序数组中找到第K个元素，为了加快搜索的速度，可以使用二分法，那么对谁二分呢，数组
么？其实要对K二分，意思是需要分别在 nums1 和 nums2 中查找第 K/2 个元素，注意这里由于两个数组的长度不定，所以有可能某个数组没有第 K/2 个数字，所以需
要先 check 一下，数组中到底存不存在第 K/2 个数字，如果存在就取出来，否则就赋值上一个整型最大值（目的是要在 nums1 或者 nums2 中先淘汰 K/2 个较小的
数字，判断的依据就是看 midVal1 和 midVal2 谁更小，但如果某个数组的个数都不到 K/2 个，自然无法淘汰，所以将其对应的 midVal 值设为整型最大值，以保证
其不会被淘汰），若某个数组没有第 K/2 个数字，则淘汰另一个数组的前 K/2 个数字即可。举个例子来说吧，比如 nums1 = {3}，nums2 = {2, 4, 5, 6, 7}，
K=4，要找两个数组混合中第4个数字，则分别在 nums1 和 nums2 中找第2个数字，而 nums1 中只有一个数字，不存在第二个数字，则 nums2 中的前2个数字可以直接
跳过，为啥呢，因为要求的是整个混合数组的第4个数字，不管 nums1 中的那个数字是大是小，第4个数字绝不会出现在 nums2 的前两个数字中，所以可以直接跳过。

有没有可能两个数组都不存在第 K/2 个数字呢，这道题里是不可能的，因为K不是任意给的，而是给的 m+n 的中间值，所以必定至少会有一个数组是存在第 K/2 个数字
的。最后就是二分法的核心啦，比较这两个数组的第 K/2 小的数字 midVal1 和 midVal2 的大小，如果第一个数组的第 K/2 个数字小的话，那么说明要找的数字肯定
不在 nums1 中的前 K/2 个数字，可以将其淘汰，将 nums1 的起始位置向后移动 K/2 个，并且此时的K也自减去 K/2，调用递归，举个例子来说吧，比如 nums1 = 
{1, 3}，nums2 = {2, 4, 5}，K=4，要找两个数组混合中第4个数字，那么分别在 nums1 和 nums2 中找第2个数字，nums1 中的第2个数字是3，nums2 中的第2个数
字是4，由于3小于4，所以混合数组中第4个数字肯定在 nums2 中，可以将 nums1 的起始位置向后移动 K/2 个。反之，淘汰 nums2 中的前 K/2 个数字，并将 nums2 
的起始位置向后移动 K/2 个，并且此时的K也自减去 K/2，调用递归即可，参见代码如下：

```
public static double findMedianSortedArrays_DC(int[] nums1, int[] nums2) {
    int m = nums1.length, n = nums2.length;
    int left = (m + n + 1) / 2, right = (m + n + 2) / 2;
    return (findKth(nums1, 0, nums2, 0, left) + findKth(nums1, 0, nums2, 0, right)) / 2.0;
}
//寻找两个数组合并后的第k个数
public static int findKth(int[] nums1, int i, int[] nums2, int j, int k) {
    if (i >= nums1.length) return nums2[j + k - 1];
    if (j >= nums2.length) return nums1[i + k - 1];
    if (k == 1) return Math.min(nums1[i], nums2[j]);
    int midVal1 = (i + k / 2 - 1 < nums1.length) ? nums1[i + k / 2 - 1] : Integer.MAX_VALUE;
    int midVal2 = (j + k / 2 - 1 < nums2.length) ? nums2[j + k / 2 - 1] : Integer.MAX_VALUE;
    if (midVal1 < midVal2) return findKth(nums1, i + k / 2, nums2, j, k - k / 2);
    else return findKth(nums1, i, nums2, j + k / 2, k - k / 2);
}
```

## 排序算法（堆排序和快速排序）

### 堆排序
#### 堆
二叉堆可分为两种形式：最大堆（根节点的值大于其子节点的值）和最小堆（根节点的值小于其子节点的值），可以完全二叉树或一维数组的方式来表示。最小堆通常用于
构造优先队列
#### 维护堆的性质（时间复杂度O(logn)）
在堆中添加或者删除数据的时候，我们需要维持堆的基本性质
maxHeapify是维护最大堆性质的重要过程，它输入一个数组A和一个下标i。调用该方法的时候，我们假定left(i)和right(i)的二叉树都是最大堆，但这时A[i]有可能
小于其孩子，该方法通过让A[i]的值在最大堆中“逐级下降”，从而使得以下标i为根节点的子树重新遵循最大堆的性质
```
//维持堆的性质(在数组中，下标需要减1。在数组中以下标来代表树中的下标时，需要从1开始)
//i代表该元素在树中的位置（根节点的索引为1）
//length代表有效元素的个数
public static void maxHeapify(int[] A, int i, int length) {
    int left = left(i);
    int right = right(i);
    int largest = i;
    if (left <= length && A[left - 1] > A[i - 1]) largest = left;
    if (right <= length && A[right - 1] > A[largest - 1]) largest = right;
    if (largest != i) {
        A[i - 1] = A[i - 1] + A[largest - 1];
        A[largest - 1] = A[i - 1] - A[largest - 1];
        A[i - 1] = A[i - 1] - A[largest - 1];
        maxHeapify(A, largest, length);
    }
}
```
#### 构建堆（时间复杂度O(n)）
可以使用自底向上的方法利用maxHeapify方法把一个大小为n=A.length的数组A转换为最大堆。因为子数组A[n/2..n-1]中的元素都是树的叶子结点。每个叶子结点都可
以看成只包含一个元素的堆。方法buildMaxHeap对树中的其他节点都调用一次maxHeapify方法。也就是自底向上对每一个非叶子结点都调整一下其结构，使其满足最大
堆的性质
```
//构建堆
public static void buildMaxHeap(int[] A) {
    for (int i = A.length / 2; i >= 1; i--) maxHeapify(A, i, A.length);
}
```
#### 堆排序
初始时候，堆排序算法利用buildMaxHeap将输入数组A建成最大堆。因为数组中的最大元素总在根节点A[1]中，通过把它与A[n]互换，我们可以让元素放到正确的位置。
这时，如果我们从堆中去掉节点n，剩余的节点中，原来根的孩子节点任然是最大堆，但是新的根节点可能会违背最大堆的性质，所以我们要调用maxHeapify(A,1)，重新
构造一个新的最大堆。堆排序算法会不断重复这一过程，直到堆的大小从n-1降到2。
```
//堆排序
public static int[] heapSort(int[] A) {
    List<Integer> ret = new ArrayList<>();
    int validLength = A.length;
    for (int i = A.length; i >= 2; i--) {
        A[0] = A[0] + A[i - 1];
        A[i - 1] = A[0] - A[i - 1];
        A[0] = A[0] - A[i - 1];
        maxHeapify(A, 1, --validLength);
    }
    return A;
}
```
