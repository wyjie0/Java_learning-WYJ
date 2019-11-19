# Algorithm
算法学习（java）
根据自己学习《算法导论》的顺序，每一部分附加一些LeetCode上面的题目（也会从大牛那儿借鉴内容），记录自己算法的学习过程。
* [分治策略](https://github.com/wyjie0/Algorithm/issues/1)
## 排序算法（堆排序和快速排序）
* [堆排序](https://github.com/wyjie0/Algorithm/issues/6)
* [快速排序](https://github.com/wyjie0/Algorithm/issues/7)
## 数据结构（二叉搜索树和红黑树）
* [二叉搜索树](https://github.com/wyjie0/Algorithm/issues/8)
* [红黑树](https://github.com/wyjie0/Algorithm/issues/9)
### 红黑树
#### 红黑树的性质
红黑树是一颗二叉搜索树，它在每个结点上增加了一个存储位来表示结点的颜色，可以是RED或BLACK。通过对任何一条**从根到叶子结点**的简单路径上**各个结点**的**颜色**进行约束，红黑树确保没有一条路径会比其他路径长出2倍，因而是近似于**平衡的**。
树中每个结点包含5个属性：color，key，left，right和parent。如果一个结点没有子结点或父结点，则该结点相应指针属性的值为null。把这些**null**视为**叶节点**（外部结点），把带关键字的结点视为树的内部节点
一颗红黑树是满足下面**红黑性质**的二叉搜索树：
1. 每个结点或是红色的，或是黑色的
2. 根结点是黑色的
3. 每个叶结点（null）是黑色的
4. 如果一个结点是红色的，则它的两个子结点都是黑色的
5. 对每个结点，从该结点到其所有后代叶结点的简单路径上，均包含相同数目的黑色结点
