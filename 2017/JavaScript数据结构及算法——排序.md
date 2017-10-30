---
layout: post
title:  "JavaScript数据结构及算法——排序"
date:   2017-10-22 20:13:00
categories: 数据结构及算法
tags: JavaScript
author: 薛彬
---

* content
{:toc}

本文主要记录的是JavaScript实现常用的排序算法，冒泡排序、快速排序、归并排序等。




## 前言

用JavaScript写算法是种怎么样的体验？不喜欢算法的我最近也对数据结构和算法有点兴趣。。。所以，将会有这些：

- JavaScript数据结构及算法——栈
- JavaScript数据结构及算法——队列
- JavaScript数据结构及算法——链表
- [JavaScript数据结构及算法——排序](https://github.com/axuebin/articles/issues/12)
- [JavaScript数据结构及算法——查找](https://github.com/axuebin/articles/issues/13)
- JavaScript数据结构及算法——树

现阶段我对于数据结构、算法的理解还很浅，希望各位大佬多多指导。

## 排序

> 介绍排序算法

## 冒泡排序

说到冒泡排序，大家都很熟悉，顾名思义，是一种“冒泡”的过程。

**主要思想**：比较任何两个相邻的项，如果第一个比第二个大，则交换它们。

**时间复杂度**：O(n2)

**空间复杂度**：O(1)

如何实现呢？是不是遍历所有需要排序的数据，然后将它和所有数比较一次，然后就可以了？

道理是有的，我们试试看：

```javascript
function bubbleSort(arr) {
  const len = arr.length; // 声明一个len来存储数组的长度
  let temp = 0;
  for (let i = 0; i < len; i += 1) { // 外循环遍历数组
    for (let j = 0 ; j < len - 1 ; j += 1) { // 内循环执行当前项和下一项进行比较
      if (arr[j] > arr[j + 1]) {  // 如果当前项比下一项大，则交换它们
        temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
      console.log(arr);
    }
  }
  return arr;
}
```

我们通过输出数组来看一下整个流程：

```javascript
[5, 4, 3, 2, 1] 
[4, 5, 3, 2, 1] // 5>4，交换
[4, 3, 5, 2, 1] // 5>3，交换
[4, 3, 2, 5, 1] // 5>2，交换
[4, 3, 2, 1, 5] // 5>1，交换
[3, 4, 2, 1, 5] // 4>3，交换
[3, 2, 4, 1, 5] // 4>2，交换
[3, 2, 1, 4, 5] // 4>1，交换
[3, 2, 1, 4, 5] // 4<5，不交换
[2, 3, 1, 4, 5] // 3>2，交换
[2, 1, 3, 4, 5] // 3>1，交换
[2, 1, 3, 4, 5] // 3<4，不交换
[2, 1, 3, 4, 5] // 4<5，不交换
[1, 2, 3, 4, 5] // 2>1，交换
[1, 2, 3, 4, 5] // 2<3，不交换
[1, 2, 3, 4, 5] // 3<4，不交换
[1, 2, 3, 4, 5] // 4<5，不交换
[1, 2, 3, 4, 5] // 1<2，不交换
[1, 2, 3, 4, 5] // 2<3，不交换
[1, 2, 3, 4, 5] // 3<4，不交换
[1, 2, 3, 4, 5] // 4<5，不交换
```

排序确实是排好了，但是我们发现，有很多的不必要的比较，我们应该想办法避免这些。想一想，这些都是在内循环中对已经排序过的数进行比较，所以我们可以稍稍改进一下代码：

```javascript
function bubbleSort(arr) {
  const len = arr.length;
  let temp = 0;
  for (let i = 0; i < len; i += 1) {
    for (let j = 0 ; j < len - 1 - i ; j += 1) {
      if (arr[j] > arr[j + 1]) {
        temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
      console.log(arr);
    }
  }
  return arr;
}
```

在内循环中，我们另 `j` 的取值到 `len-1-i` 为止，因为再往后的数已经排序好了。同样地，我们来看看流程：

```javascript
[5, 4, 3, 2, 1] 
[4, 5, 3, 2, 1] // 5>4，交换
[4, 3, 5, 2, 1] // 5>3，交换
[4, 3, 2, 5, 1] // 5>2，交换
[4, 3, 2, 1, 5] // 5>1，交换
[3, 4, 2, 1, 5] // 4>3，交换
[3, 2, 4, 1, 5] // 4>2，交换
[3, 2, 1, 4, 5] // 4>1，交换
[2, 3, 1, 4, 5] // 3>2，交换
[2, 1, 3, 4, 5] // 3>1，交换
[1, 2, 3, 4, 5] // 2>1，交换
```

nice，没必要的比较已经完全没有了。

## 选择排序

**主要思想**：找到数组中的最小值然后将其放置在第一位，接着第二位第三位。。。

**时间复杂度**：O(n2)

**空间复杂度**：O(1)

直接看代码吧：

```javascript
function selectionSort(arr) {
  const len = arr.length; // 用len存储数组长度
  let indexMin = 0; // 最小值索引
  let temp = 0;
  for (let i = 0; i < len - 1; i += 1) { //外循环遍历数组
    indexMin = i; // 先假设这一轮循环的第一个值是最小的
    for (let j = i; j < len; j += 1) { // 比较i时候会比它之后的数小，如果小，则令indexMin存储这个更小值的索引
       if (arr[indexMin] > arr[j]) {
        indexMin = j;
      }
    }
    if (i !== indexMin) { // 执行完内循环之后判断当前值i是否是最小的，如果不是，就要交换
      temp = arr[i];
      arr[i] = arr[indexMin];
      arr[indexMin] = temp;
    }
    console.log(arr);
  }
  return arr;
}
```

```javascript
[5, 4, 3, 2, 1] 
[1, 4, 3, 2, 5] // 寻找最小值1，交换1和5
[1, 2, 3, 4, 5] // 寻找最小值2，交换2和4
[1, 2, 3, 4, 5] // 寻找最小值3，不交换
[1, 2, 3, 4, 5] // 寻找最小值4，不交换
[1, 2, 3, 4, 5] // 寻找最小值5，不交换
```

是不是很酷，然而它的时间复杂度其实还是 `O(n2)`。

## 插入排序

**主要思想**：每次将一个元素与已排序的元素进行逐一比较，直到找到合适的位置按大小插入。

**时间复杂度**：O(n2)

**空间复杂度**：O(1)

直接看代码吧：

```javascript
function insertionSort(arr) {
  const len = arr.length; // 数组长度
  let j = 0; // 使用的辅助变量
  let temp = 0;
  for (let i = 1; i < len; i++) { // 外循环，从1开始
    j = i; // 当前索引赋给j
    temp = arr[i]; // 当前值存在temp
    while (j > 0 && arr[j - 1] > temp) { // 如果j前面的数比它大，就往前移，直到第一位
      arry[j] = arr[j - 1];
      j--;
    }
    arr[j] = temp; // temp是要排序的那个数，放到正确的j的位置上
  }
  return arr;
}
```

## 归并排序

**主要思想**：思想主要是分治。将原始数组划分成较小的数组，直到每个小数组只有一个位置，然后将小数组归并成较大的数组。

**时间复杂度**：O(nlogn)

**空间复杂度**：O(n)

直接看代码吧：

```javascript
// 分
function mergeSort(arr) {
  const len = arr.length;
  if (len === 1) {
    return arr;
  }
  const mid = Math.floor(len / 2);
  const left = arr.slice(0, mid);
  const right = arr.slice(mid, len);
  return merge(mergeSort(left), mergeSort(right));
}
 
// 合
function merge(left, right) {
  const result = [];
  let il = 0;
  let ir = 0;
  while (il < left.length && ir < right.length) {
    if (left[il] < right[ir]) {
      result.push(left[il++]);
    } else {
      result.push(right[ir++]);
    }
  }
  while (il < left.length) {
    result.push(left[il++]);
  }
  while (ir < right.length) {
    result.push(right[ir++]);
  }
  return result;
}
```

## 快速排序

来看看面试中最喜欢考察的快速排序。

**主要思想**：每次将一个元素与已排序的元素进行逐一比较，直到找到合适的位置按大小插入。

**时间复杂度**：O(nlogn)

**空间复杂度**：O(logn)

```javascript
function quickSort(arr) {
  if (arr.length <= 1) {
    return arr;
  }
  const pivotIndex = Math.floor(arr.length / 2);
  const pivot = arr.splice(pivotIndex, 1)[0]; // 将这个元素取出并从原数组中删除
  const left = [];
  const right = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] < pivot) {
      left.push(arr[i]);
    } else {
      right.push(arr[i]);
    }
  }
  return quickSort(left).concat(pivot, quickSort(right));
}
```
