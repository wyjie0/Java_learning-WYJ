# Algorithm
算法学习（java）
根据自己学习《算法导论》的顺序，每一部分附加一些LeetCode上面的题目（也会从大牛那儿借鉴内容），记录自己算法的学习过程。
* [分治策略](https://github.com/wyjie0/Algorithm/issues/1)
## 排序算法（堆排序和快速排序）
* [堆排序](https://github.com/wyjie0/Algorithm/issues/6)
* [快速排序](https://github.com/wyjie0/Algorithm/issues/7)
算法的关键部分就在于对数组的划分，它实现了对子数组A[p..r]的原址重排
```
public static int partition(int[] A, int p, int r) {
    int pivot = A[r];
    int i = p - 1;
    //0--i的数都为小于pivot的数，i+1--r的数都为大于pivot的数
    for (int j = p; j < r; j++) {
        //i代表比pivot小的数的个数，如果遇到比pivot小的数，i就递增
        if (A[j] <= pivot) {
            i++;
            int temp = A[i];
            A[i] = A[j];
            A[j] = temp;
        }
    }
    int temp = A[i + 1];
    A[i + 1] = A[r];
    A[r] = temp;
    return i + 1;
}
```
在一个样例数组上的partition操作过程如下：
<img src="https://github.com/wyjie0/Algorithm/blob/master/img/quickSort1.png" width="200px">
