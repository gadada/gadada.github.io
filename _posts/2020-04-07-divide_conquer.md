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

思路就是通过二分查找枚举mindist，从而找到最大的那一个

另外，枚举之前要先排序，二分查找必须保证数据是有序的
```c
#include<stdio.h>
#include<stdlib.h>

int n, c;
int a[100000];
int compare(const void* a, const void* b)
{
	return *(int*)a - *(int*)b;
}
int solve(int p)
{
	int cnt = 1, last = a[0];
	for (int i = 1; i < n; ++i) {
		if ((a[i] - last) >= p) {
			++cnt;
			last = a[i];
		}
	}
	if (cnt >= c)return 1;
	else return 0;
}
int main()
{
	if (~scanf("%d %d", &n, &c)) {
		for (int i = 0; i < n; ++i)
			scanf("%d", &a[i]);
		qsort(a, n, sizeof(int), compare);
		//枚举mindist
		int r = a[n - 1] - a[0];
		int l = 1;
		while (l <= r) {
			int mid = (l + r) / 2;
			if (solve(mid))
				l = mid + 1;
			else
				r = mid - 1;
		}
		printf("%d\n", r);
	}

	return 0;
}

```
官方答案
- `int R = x[N-1] / C + 1;`缩小了初始右边界

### Q2
[派](http://cxsjsxmooc.openjudge.cn/2020t2springall/008/)

我的生日要到了！根据习俗，我需要将一些派分给大家。我有N个不同口味、不同大小的派。有F个朋友会来参加我的派对，每个人会拿到一块派（必须一个派的一块，不能由几个派的小块拼成；可以是一整个派）。

我的朋友们都特别小气，如果有人拿到更大的一块，就会开始抱怨。因此所有人拿到的派是同样大小的（但不需要是同样形状的），虽然这样有些派会被浪费，但总比搞砸整个派对好。当然，我也要给自己留一块，而这一块也要和其他人的同样大小。

请问我们每个人拿到的派最大是多少？每个派都是一个高为1，半径不等的圆柱体。

- 输入：
  第一行包含两个正整数N和F，1 ≤ N, F ≤ 10 000，表示派的数量和朋友的数量。
  第二行包含N个1到10000之间的整数，表示每个派的半径。
- 输出：
  输出每个人能得到的最大的派的体积，精确到小数点后三位。
- 样例输入
  ```markdown
  3 3
  4 3 3
  ```
- 样例输出
  ```markdown
  25.133
  ```

同样是用二分查找枚举合适的答案，但是要注意这里的左边界`l`要从0开始而不是最小的蛋糕体积
```c
#include<stdio.h>
#include<stdlib.h>
#include<math.h>
#define PI acos(-1)
int n, f, p;
double a[10000];
int solve(double t)
{
	int cnt = 0;
	for (int i = 0; i < n; ++i)
		cnt += (int)(a[i] / t);

	return cnt > f;
}
int main()
{
	if (~scanf("%d %d", &n, &f)) {
		double r = 0, l = 0;
		for (int i = 0; i < n; ++i) {
			scanf("%d", &p);
			a[i] = PI * p * p;
			if (a[i] > r)r = a[i];
		}
		while (l <= r) {
			double mid = (l + r) / 2;
			if (solve(mid))
				l = mid + 1e-5;
			else
				r = mid - 1e-5;
		}
		printf("%.3lf\n", r);
	}

	return 0;
}
```