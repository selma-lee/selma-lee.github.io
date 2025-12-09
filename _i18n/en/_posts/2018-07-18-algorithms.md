---
layout: post
title: Study Tips for the Algorithms Section of the JavaScript Description of Data Structures and Algorithms
date: 2018-07-18 22:49:20
tags: [Algorithm]
categories: algorithm
---

## Introduction

I recently read "JavaScript Description of Data Structures and Algorithms", and apart from some code errors in the book, it was a book that gave me something to work with. Here I will talk about some of the algorithms mentioned in the book and a little bit of my own understanding, I hope that in the future I can skillfully use these algorithms to solve more complex problems.

<!-- more -->

## Sorting algorithms ##

### Rapid sort

Quick sort is one of the fastest sorting algorithms for handling large data sets. Many sorting functions in programming languages use quick sort, such as `Array.sort()` in Chrome's JavaScript engine V8, which uses a combination of quick sort and insertion sort.
The algorithm for quick sort is briefly described:

1. determine a "pivot point" (pivot)
2. Place all values less than the pivot point on the left side of the point, and all values greater than the pivot point on the right side of the point.
3. Repeat this process for the left and right sides until all sorting is complete.

The example below:
! [Quick Sort](/assets/img/2018/07/quickSort-1.jpeg)
Using the idea of recursion it can be realized like this with time complexity O(nlog2n)

``` js
function quickSort(arr) {
  if (arr.length <= 1) {
    return arr
  }
  const pivot = Math.floor(arr.length / 2)
  const left = []
  const right = []
  for (let i = 0, len = arr.length; i < len; i++) {
    if (arr[i] < arr[pivot]) {
      left.push(arr[i])
    } else if (arr[i] >= arr[pivot] && i !== pivot) {
      right.push(arr[i])
    }
  }
  return quickSort(left).concat(arr[pivot], quickSort(right))
}
// 调用
const arr = [2,1,4,5,8,3]
quickSort(arr, 0, arr.length - 1)
```

However, this implementation declares a number of arrays, which has a higher space complexity and memory footprint.
Here is a better implementation of sort-in-place.
Let's start with the motion picture
! [Quick Sort](/assets/img/2018/07/quickSort-2.gif)
Specific implementation:

``` js
function partition(arr, left, right) {
  const pivot = arr[left] // 支点
  let p = left
  for (let i = left + 1; i <= right; i++) {
    if (arr[i] < pivot) {
      swap(arr[++p], arr[i])
    }
  }
  swap(pivot, arr[p])
  return p // 返回支点所在位置
}
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (arr.length == 1) {
    return arr
  }
  if (left < right) {
    const p = partition(arr, left, right)
    quickSort(arr, left, p - 1)
    quickSort(arr, p + 1, right)
  }
}
// 交换函数
function swap (a, b) {
  let temp = b
  b = a
  a = temp
}
```

## Retrieval algorithm

### Sequential search

For finding data, sequential lookup is the easiest method to understand, and also belongs to one of the violent lookup techniques, suitable for arrays with elements randomly arranged.
The specific implementation is as follows, with time complexity O(n)

``` js
function seqSearch(arr, data) {
  for (let i = 0, len = arr.length; i < len; i++) {
    if (arr[i] === data) {
      return i
    }
  }
  return -1
}
```

Because all elements in the data structure may be accessed when performing a lookup, this method of lookup is less efficient, especially with larger amounts of data.
One optimized approach is to use self-organizing data. This strategy is specifically: minimize the number of lookups by placing frequently looked-up elements at the beginning of the dataset. We can have the program automatically organize the data as it runs, as in the following example:

``` js
// 对于未排序数组，使用自组织数据，并通过简单的顺序查找快速找到元素
function seqSearch(arr, data) {
  for (let i = 0, len = arr.length; i < len; i++) {
    if (arr[i] === data) {
      if (i > 0) {
        swap(arr[i], arr[i - 1]) // 查找到的元素向前移动一位，逐渐将经常查找的元素移到最前
        return i - 1
      }
    }
  }
  return -1
}
```

### Dichotomous search algorithm ###

And for ordered data, bisection lookup algorithm is more efficient than sequential lookup algorithm.
Simple description of bisection lookup algorithm:

1. set the first position of the array to the lower boundary and the last element to the upper boundary
2. if the lower boundary is less than or equal to the upper boundary, set the midpoint (mid) to (upper boundary + lower boundary) / 2
3. if mid is less than the query value, set the lower boundary to mid + 1, repeat steps 2 and 3; if mid is greater than the query value, set the upper boundary to mid - 1, repeat steps 2 and 3; otherwise, return mid

It is implemented as follows with time complexity O(log2n).

``` js
function binSearch(arr, data) {
  const upperBound = arr.length - 1
  const lowerBound = 0
  while(lowerBound <= upperBound) {
    const mid = Math.floor((lowerCound + upperBound) / 2)
    if (data > arr[mid]) {
      lowerBound = mid + 1
    } else if (data < arr[mid]) {
      upperBound = mid - 1
    } else {
      return mid
    }
  }
  return -1
}
```

For unordered datasets with frequent lookups, you can perform a quick sort and then implement the binary lookup algorithm.

### Array de-duplication

We often encounter situations where we have to de-weight an array. The easiest to understand implementation is to traverse the array and perform a sequential lookup. The implementation is as follows, with time complexity O(n^2).

``` js
function unique(arr) {
  let uqArr = []
  for (let i = 0, len = arr.length; i < len; i++) {
    if (uqArr.indexOf(arr[i]) === -1) {
      uqArr.push(arr[i])
    }
  }
  return uqArr
}
```

A lower time complexity implementation is to utilize a hash table structure, but with a higher space complexity, trading space for time. The specific implementation is as follows, with time complexity O(n).

``` js
function uniqueHash(arr) {
  let hash = {}
  let uqArr = []
  let item
  for (let i = 0, len = arr.length; i < len; i++) {
    item = arr[i]
    if(!hash[item]) {
      hash[item] = true
      uqArr.push(item)
    }
  }
  return uqArr
}
```

Building on the above, the open chaining method is used to distinguish between data types, solving the problem if an element of the array is `__proto___` and collisions.

``` js
function uniqueHash2(arr) {
  let hash = Object.create(null)
  let uqArr = []
  let item
  for (let i = 0, len = arr.length; i < len; i++) {
    item = arr[i]
    if(!hash[item][typeof item]) {
      hash[item][typeof item] = true
      uqArr.push(item)
    }
  }
  return uqArr
}
```

## Advanced algorithms

### Dynamic programming ###

We can understand dynamic programming by comparing it with recursion:

* Recursion: start from the top to decompose the problem, by solving all the decomposition out of small problems, to solve the whole problem
* Dynamic Programming: start solving the problem from the bottom, solve all the small problems, and then merge them into an overall solution to solve the whole big problem.

Look at a simple example below:
To implement the Fibonacci series (0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55...), using the idea of recursion it can be implemented like this:

``` js
function recurFib(n) {
  if (n < 2) {
    return n
  } else {
    return recurFib(n - 1) + recurFib(n - 2)
  }
}
```

This is one of the easier implementations to understand. However, we will find that too many values are recalculated in the recursive call, and when the n to be calculated is large, the implementation is inefficient.
At this point, we can find that this "big problem" can be solved by solving a number of "small problems" to solve. There is a fixed formula for solving "small problems", which is: `Fib(n) = Fib(n - 1) + Fib(n - 2)`. From there, I thought of using dynamic programming to realize it:

``` js
function dynFib(n) {
  if (n < 3) {
    const fib = n < 2 ? n : 1
    return fib
  } else {
    const val = new Array(n + 1)
    val[1] = 1
    val[2] = 1
    for (let i = 3; i <= n; i++) {
      val[i] = val[i - 1] + val[i - 2]
    }
    return val[n]
  }
}
```

This implementation saves the intermediate results with an array, which is more efficient.
In the future, when you encounter a problem that can be solved with recursion, you may consider trying to solve it with dynamic programming if there is a subproblem structure.

### The greedy algorithm ##

The greedy algorithm's strategy is to always choose the optimal solution for the moment, without considering whether this choice will have an impact on future choices. By making a series of locally "optimal" choices, it is possible to arrive at an overall "optimal" choice, or a "suboptimal" choice.
Here is an example of making change at a store, where the optimal solution is to minimize the number of coins used.

``` js
function makeChange(money) {
  let coins = []
  if (money % .25 < money) {
    coins[3] = parseInt(money / .25)
    money = money % .25
  }
  if (money % .1 < money) {
    coins[2] = parseInt(money / .1)
    money = money % .1
  }
  if (money % .05 < money) {
    coins[1] = parseInt(money / .05)
    money = money % .05
  }
  coins[0] = parseInt(money / .01)

  console.log(coins[0] + '个1美分')
  console.log(coins[1] + '个5美分')
  console.log(coins[2] + '个10美分')
  console.log(coins[3] + '个25美分')
}
```

In this case, this scheme always finds the optimal solution. However, if the denomination of the coin is changed to: .25, .6, .5, .3, .1. When 1 is encountered, according to the greedy algorithm it will decompose to `.6 + .3 + .1`, and the optimal solution should be `.5 * 2`.
Here are just a few simple examples. There are many more applications of dynamic programming and greedy algorithms, so I won't describe them all here.

## Reference

* JavaScript Description of Data Structures and Algorithms
* [JavaScript Topic of Interpreting v8 sort source code](https://github.com/mqyqingfeng/Blog/issues/52)
* v8 array source code (https://github.com/v8/v8/blob/master/src/js/array.js)
