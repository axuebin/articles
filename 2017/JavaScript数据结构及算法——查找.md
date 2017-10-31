本文主要记录的是JavaScript实现常用的查找算法。

## 前言

用JavaScript写算法是种怎么样的体验？不喜欢算法的我最近也对数据结构和算法有点兴趣。。。所以，将会有这些：

- JavaScript数据结构及算法——栈
- JavaScript数据结构及算法——队列
- JavaScript数据结构及算法——链表
- [JavaScript数据结构及算法——排序](https://github.com/axuebin/articles/issues/12)
- [JavaScript数据结构及算法——查找](https://github.com/axuebin/articles/issues/13)
- JavaScript数据结构及算法——树

现阶段我对于数据结构、算法的理解还很浅，希望各位大佬多多指导。

## 查找

> 查找是在大量的信息中寻找一个特定的信息元素，在计算机应用中，查找是常用的基本运算，例如编译程序中符号表的查找。本文简单概括性的介绍了常见的七种查找算法，说是七种，其实二分查找、插值查找以及斐波那契查找都可以归为一类——插值查找。插值查找和斐波那契查找是在二分查找的基础上的优化查找算法。

这里主要提到如何用JavaScript实现顺序查找和二分查找。

## 顺序查找

**主要思想**：将每一个数据结构中的元素和要查找的元素做比较，类似于JavaScript中indexOf

**时间复杂度**：O(n)

代码：

```javascript
function sequentialSearch(array,item){
  for (let i = 0; i < array.length; i += 1) {
    if ( item === array[i] ) {
      return i;
    }
  }
  return -1;
}
```

比如我现在有这样一个数组 `[5, 4, 3, 2, 1]` ，然后我们需要在其中找到 `3` ，整个流程应该是这样：

```javascript
[5, 4, 3, 2, 1] // 5 !== 3，继续遍历
[5, 4, 3, 2, 1] // 4 !== 3，继续遍历
[5, 4, 3, 2, 1] // 3 === 3，找到了
```

## 二分查找

**主要思想**：首先这个数组是排好序的，然后将数组一直二分缩小范围，直到找到为止。

**时间复杂度**：O(logn)

代码：

```javascript
function binarySearch(array, item) {
  const sortArray = quickSort(array); // 对数组进行快排
  let low = 0; // 设置左边界
  let high = sortArray.length - 1; // 设置右边界
  let mid = 0; // 设置中间值
  let element = 0;
  while (low < high) {
    mid = Math.floor((low + high) / 2); // 选择整个数组的中间值
    element = sortArray[mid];
    if (element < item) { // 如果待搜索值比选中值要大，则返回步骤一在右边的字数组中寻找
      low = mid + 1;
    } else if (element > item) { // 如果待搜索值比选中值要小，则返回步骤一在左边的字数组中寻找
      high = mid - 1;
    } else {
      return mid; // 如果刚好选中，恭喜你，直接返回
    }
  }
  return -1;
}
```