---
title: "算法基础part3"
tags:
  - C/C++
  - Binary Search
  - Divide and Conquer
categories:
  - Algorithm Basics
---
数学归纳法、递归和无穷递降是三种常见的数学论证方法，递归是在分治法思想指导下的一种实现方法。

先看一段伪代码：
```markdown
int gcd(int n, int m)
{
    if(m == 0)
        return n;
    else
        return gcd(m, n % m);
}
```
这是求最大公约数的[辗转相除法](https://zh.wikipedia.org/wiki/%E8%BC%BE%E8%BD%89%E7%9B%B8%E9%99%A4%E6%B3%95)算法，其中分治法的思想：

- 确定基线条件(a能够被b整除)
- 不断将问题进行分解，或者缩小问题规模，直到符合上面的基线条件

## 分治
分治法是建基于多项分支递归的一种很重要的算法范式。字面上的解释是“分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。

### Q1
[Aggressive cows](http://cxsjsxmooc.openjudge.cn/2020t2springall/007/)

Farmer John has built a new long barn, with N (2 <= N <= 100,000) stalls. The stalls are located along a straight line at positions x1,...,xN (0 <= xi <= 1,000,000,000).

His C (2 <= C <= N) cows don't like this barn layout and become aggressive towards each other once put into a stall. To prevent the cows from hurting each other, FJ want to assign the cows to the stalls, such that the minimum distance between any two of them is as large as possible. What is the largest minimum distance?

- 输入：
  Line 1: Two space-separated integers: N and C

  Lines 2..N+1: Line i+1 contains an integer stall location, xi

- 输出：
  Line 1: One integer: the largest minimum distance
- 样例输入
  ```markdown
  5 3
  1
  2
  8
  4
  9
  ```
- 样例输出
  ```markdown
  3
  ```
- OUTPUT DETAILS:

  FJ can put his 3 cows in the stalls at positions 1, 4 and 8, resulting in a minimum distance of 3.

  Huge input data,scanf is recommended.

分析

