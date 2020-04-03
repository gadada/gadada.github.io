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
输出这个字符串的所有排列方式，每行一个排列。要求字母序比较小的排列在前面。字母序如下定义：

已知S = s1s2...sk , T = t1t2...tk，则S < T 等价于，存在p (1 <= p <= k)，使得
s1 = t1, s2 = t2, ..., sp - 1 = tp - 1, sp < tp成立。
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

