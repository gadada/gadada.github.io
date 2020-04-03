---
title: "算法基础part2"
tags:
  - C/C++
  - Recursion
categories:
  - Algorithm Basics
---
趁热打铁

## 递归
一个函数调用其自身的方法，可以看成是子问题的求解，但必须有终止条件

### Q1
[全排列](http://cxsjsxmooc.openjudge.cn/2020t2springall/003/)

给定一个由不同的小写字母组成的字符串，输出这个字符串的所有全排列。 我们假设对于小写字母有'a' < 'b' < ... < 'y' < 'z'，而且给定的字符串中的字母已经按照从小到大的顺序排列。
- 输入：
输入只有一行，是一个由不同的小写字母组成的字符串，已知字符串的长度在1到6之间。
- 输出：
输出这个字符串的所有排列方式，每行一个排列。要求字母序比较小的排列在前面。字母序如下定义：已知S = s1s2...sk , T = t1t2...tk，则S < T 等价于，存在p (1 <= p <= k)，使得s1 = t1, s2 = t2, ..., sp - 1 = tp - 1, sp < tp成立。
- 样例输入
```markdown
abc
```
- 样例输出
```markdown
abc
acb
bac
bca
cab
cba
```

字典序、邻位对换法、循环左移法、循环右移法、递增进位制法、递减进位制法都是常见的[全排列生成算法](https://zh.wikipedia.org/wiki/%E5%85%A8%E6%8E%92%E5%88%97%E7%94%9F%E6%88%90%E7%AE%97%E6%B3%95)。
邻位对换法简单快速，其他的需要取中介数比较繁琐，果断取舍。
- 邻位对换法

初始化n个元素的排列为123……n，并规定其元素的方向都是向左的，元素的方向用一个数组b来表示，当b[i]=0，表示第i个元素的方向向左，当b[i]=1时表示第i个元素的方向向右。
在排列中找出排列中所有处于活动状态的元素中最大的一个。
将它与它所指向相邻元素交换。
把排列中大于上面找出的处在活动状态的最大元素大的其他元素的方向倒转。
- 分析
```markdown
简单来说邻位对换法的规则就是：
    1)如果一个元素的可移动方向所指的邻位比它小，则它处于活动状态；如果可移动方向没有元素或者元素比它更大，则不处于活动状态
    2)每次先寻找最大的活动状态的元素`max`，将它与所指邻位交换，然后把所有比`max`大的元素的移动方向反转，直到所有元素都不处于活动状态为止
```