# Algorithm
算法学习（java）
根据自己学习《算法导论》的顺序，每一部分附加一些LeetCode上面的题目（也会从大牛那儿借鉴内容），记录自己算法的学习过程。
## [分治策略](https://github.com/wyjie0/Algorithm/issues/1)
## 排序算法（堆排序和快速排序）
### [堆排序](https://github.com/wyjie0/Algorithm/issues/6)
### [快速排序](https://github.com/wyjie0/Algorithm/issues/7)
#### 快速排序
对数组A[p...r]进行快速排序的三步分治过程
* 分解：数组A[p...r]被划分为两个（可能为空）子数组A[p...q-1]和A[q+1...r]，使得A[p...q-1]中的每一个元素都小于等于A[q]，而A[q]也小于等于A[q+1...r]
中的每一个元素。其中，计算下标q也是划分过程的一部分。
* 解决：通过递归调用快速排序，对子数组A[p...q-1]和A[q+1...r]进行排序
* 合并：因为子数组都是原址排序，所以不需要合并操作
```
//快速排序算法
public static void quickSort(int[] A, int p, int r) {
    if (p < r) {
        int q = partion(A, p, r);
        quickSort(A, p, q - 1);
        quickSort(A, q + 1, r);
    }
}
```
为了排序一个数组A的全部元素，起始调用是partion(A, 0, A.length - 1)
##### 数组的划分
算法的关键部分就在于对数组的划分，它实现了对子数组A[p..r]的原址重排
```
public static int partion(int[] A, int p, int r) {
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
