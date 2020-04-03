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
邻位对换法简单快速容易理解，果断取舍。
- 邻位对换法

    1）先将原排列从小到大排序，并规定所有元素的可移动方向向左

    2）如果一个元素的可移动方向所指的邻位比它小，则它处于活动状态；如果可移动方向没有元素或者元素比它更大，则不处于活动状态

    3）每次先寻找最大的活动状态的元素`max`，将它与所指邻位交换，然后把所有比`max`大的元素的移动方向反转，直到所有元素都不处于活动状态为止

```c
#include <stdio.h>
#include <string.h>

int move[6] = { 0 };
int n;
void swap(char* a, char* b)
{
	char tmp = *a;
	*a = *b;
	*b = tmp;
}
void reverse(int* t)
{
	if (*t == 0)*t = 1;
	else *t = 0;
}
void allrange(char* p)
{
	printf("%s\n", p);
	int max = 0, maxnum = -1;
	for (int i = 0; i < n; ++i)
		if (move[i] == 0 && i - 1 >= 0 && p[i - 1] < p[i] ||
			move[i] == 1 && i + 1 < n && p[i + 1] < p[i]) {
			if (p[i] > max) {
				max = p[i];
				maxnum = i;
			}
		}
	if (max == 0)return;
	if (move[maxnum] == 0) {
		swap(&p[maxnum], &p[maxnum - 1]);
		swap(&move[maxnum], &move[maxnum - 1]);
		maxnum -= 1;
		for (int i = 0; i < n; ++i)
			if (p[i] > p[maxnum])reverse(&move[i]);
	}
	else {
		swap(&p[maxnum], &p[maxnum + 1]);
		swap(&move[maxnum], &move[maxnum + 1]);
		maxnum += 1;
		for (int i = 0; i < n; ++i)
			if (p[i] > p[maxnum])reverse(&move[i]);
	}
	allrange(p);
}

int main()
{
	char ori[7];
	scanf("%s", ori);
	n = strlen(ori);
	allrange(ori);

	return 0;
}
```
将算法表示出来大概就是这个样子，然后这是输出
```markdown
123
132
312
321
231
213
```
那么问题来了，题目还要求答案按字母序从小到大打印排列，大概要把需要输出的排列存起来，花`nlog(n)`[^n]的时间排序再输出，挺浪费时间空间的，而且用到的还是尾递归，其实是可以写成循环的，那么有没有更好的方法呢？

[^n]: `nlog(n)`中`n`指的是全排列的个数，本身就是`m!`的复杂度。

- 递归实现

    从集合中依次选出每一个元素，作为排列的第一个元素，然后对剩余的元素进行全排列，将其化简成子问题，做递归处理，从而得到所有元素的全排列。在处理子问题前将其排好序就OK了

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void swap(char* a, char* b)
{
	char tmp = *a;
	*a = *b;
	*b = tmp;
}
int compare(const void* a, const void* b)
{
	return *(char*)a - *(char*)b;
}
void allrange(char* p, int from, int to)
{
	if (from == to) printf("%s\n", p);
	else {
		for (int j = from; j <= to; ++j) {
			char tmp[7];
			strcpy(&tmp[from], &p[from]);
			swap(&p[j], &p[from]);
			qsort(&p[from + 1], to - from, sizeof(char), compare);
			allrange(p, from + 1, to);
			strcpy(&p[from], &tmp[from]);
		}
	}
}

int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	char ori[7];
	scanf("%s", ori);
	int n = strlen(ori);
	allrange(ori, 0, n - 1);

	return 0;
}
```
测试6个字符1ms通过，算是完美达成目标。然后来看下官方代码
```c++
#include <iostream>
#include <algorithm>
#include <cstring>
#include <string>
using namespace std;
const int M = 8;
char str[M];
char permutation[M];
bool used[M] = {0};
int L = 0;
void Permutation(int n)
{
	if( n == L ) {
		permutation[L] = 0;
		cout << permutation << endl;
		return ;
	}
	for(int i = 0;i < L; ++i) {
		if( !used[i]) {
			used[i] = true;
			permutation[n] = str[i];
			Permutation(n+1);
			used[i] = false;
		}
	}
}
int main()
{
	cin >> str;
	L = strlen(str);
	sort(str,str+L);
	Permutation(0);
	return 0;
}
```
。。。