# Algorithm
算法学习（java）
根据自己学习《算法导论》的顺序，每一部分附加一些LeetCode上面的题目（也会从大牛那儿借鉴内容），记录自己算法的学习过程。
## [分治策略](https://github.com/wyjie0/Algorithm/issues/1)

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
