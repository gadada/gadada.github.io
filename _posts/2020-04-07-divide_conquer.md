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

官方答案
```c++
#include <iostream>
#include <algorithm>
#include <iomanip>
#include <cmath>
using namespace std;
//const double PI = 3.1415927;
//PI取3.1415926都不行，结果偏小， PI取3.1415927可以
const double PI=acos(-1.0);
const double eps = 1e-6;
int r[10010];
int N,F;
bool Valid(double V)
{
	if( V < eps )
		return true;
	int total = 0;
	for(int i = 0;i < N; ++i) {
		double n =  r[i]*r[i] / V;
		total += n;
		if( total >= F)
			return true;
	}
	return false;
}
int main()
{
	cin >> N >> F;
	++F;
	double maxV = 0;
	for(int i = 0;i < N; ++i) {
		cin >> r[i];
		maxV = max(maxV,(double)r[i]*r[i]);
	}
	double L = 0,R = maxV;
	while( R - L > eps ) {
		double midV = L + (R-L )/2;
		if( Valid(midV) )
			L = midV;
		else
			R = midV;
	}
	cout << fixed << setprecision(3) << PI * L ;
	return 0;
}
```

### Q3
[月度开销](http://cxsjsxmooc.openjudge.cn/2020t2springall/009/)

农夫约翰是一个精明的会计师。他意识到自己可能没有足够的钱来维持农场的运转了。他计算出并记录下了接下来 N (1 ≤ N ≤ 100,000) 天里每天需要的开销。

约翰打算为连续的M (1 ≤ M ≤ N) 个财政周期创建预算案，他把一个财政周期命名为fajo月。每个fajo月包含一天或连续的多天，每天被恰好包含在一个fajo月里。

约翰的目标是合理安排每个fajo月包含的天数，使得开销最多的fajo月的开销尽可能少。



- 输入：
  第一行包含两个整数N,M，用单个空格隔开。
  接下来N行，每行包含一个1到10000之间的整数，按顺序给出接下来N天里每天的开销。
- 输出：
  一个整数，即最大月度开销的最小值。
- 样例输入
  ```markdown
  7 5
  100
  400
  300
  100
  500
  101
  400
  ```
- 样例输出
  ```markdown
  500
  ```
- 提示

  若约翰将前两天作为一个月，第三、四两天作为一个月，最后三天每天作为一个月，则最大月度开销为500。其他任何分配方案都会比这个值更大。

分析
- 左值`l`为所有天数里最大花销的一天
- 右值`r`则为所有天数花销的总和

```c
#include<stdio.h>

int n, m;
int a[100000];
int solve(int t) // t > a[i]
{
	int sum = 0, cnt = 0, i = 0;
	while (i < n) {
		if (sum + a[i] > t) {
			sum = 0;
			cnt++;
		}
		else
			sum += a[i++];
	}
	if (sum != 0) cnt++;
	if (cnt > m)return 0;
	else return 1;
}
int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	if (~scanf("%d %d", &n, &m)) {
		int l = 0, r = 0;
		for (int i = 0; i < n; ++i) {
			scanf("%d", &a[i]);
			if (a[i] > l)l = a[i];
			r += a[i];
		}
		while (l <= r) {
			int mid = (l + r) / 2;
			if (solve(mid))
				r = mid - 1;
			else
				l = mid + 1;
		}
		printf("%d\n", l);
	}

	return 0;
}
```

官方答案
```c++
#include <iostream>
#include <algorithm>
#include <iomanip>
#include <cmath>
using namespace std;
int N,M ;
int cost[100100];
bool Valid(int c)
{
	int m = 1; //总月数
	int curCost = 0; //本月花销
	for(int i = 0;i < N; ++i) {
		if(cost[i] > c )
			return false;
		if( curCost + cost[i] > c ) {
			curCost = cost[i];
			++m;
			if( m > M )
				return false;
		}
		else
			curCost += cost[i];
	}
	return true;
}
int main()
{
	cin >> N >> M;
	int L = 1 << 30,R = 0;
	for(int i = 0; i < N; ++i) {
		cin >> cost[i];
		L = min(L,cost[i]);
		R += cost[i];
	}
	int lastValid = 0;
	while(L <= R) {
		int mid = L + (R-L)/2;
		if(Valid(mid)) {
			lastValid = mid;
			R = mid - 1;
		}
		else
			L = mid +1;
	}
	cout << lastValid ;
	return 0;
}
```

### Q4
[输出前k大的数](http://cxsjsxmooc.openjudge.cn/2020t2springall/010/)

给定一个数组，统计前k大的数并且把这k个数从大到小输出。

- 输入

  第一行包含一个整数n，表示数组的大小。n < 100000。

  第二行包含n个整数，表示数组的元素，整数之间以一个空格分开。每个整数的绝对值不超过100000000。

  第三行包含一个整数k。k < n。
- 输出

  从大到小输出前k大的数，每个数一行。
- 样例输入
  ```markdown
  10
  4 5 6 9 8 7 1 2 3 0
  5
  ```
- 样例输出
  ```markdown
  9
  8
  7
  6
  5
  ```

第一反应就是qsort排序再输出，复杂度`nlogn`
```c
#include<stdio.h>
#include<stdlib.h>
int compare(const void* a, const void* b) {
	return *(int*)b - *(int*)a;
}
int main()
{
	int n, k;
	int a[100000];
	scanf("%d", &n);
	for (int i = 0; i < n; ++i)
		scanf("%d", &a[i]);
	scanf("%d", &k);
	qsort(a, n, sizeof(int), compare);
	for(int i = 0; i < k; ++i)
		printf("%d\n", a[i]);

	return 0;
}
```
居然过了，看来给的数据没有太为难

那么怎么融入分治法的思想，有点像QuickSort

```c
#include<stdio.h>
#include<stdlib.h>
int a[100010];
void swap(int *a, int *b)
{
	int tmp = *a;
	*a = *b;
	*b = tmp;
}
int compare(const void* a, const void* b)
{
	return *(int*)a - *(int*)b;
}
void FindMaxK(int a[], int s, int e, int k)
{
	if (s - e + 1 == k)
		return;
	int pivot = a[s];
	int i = s, j = e;
	while (i != j) {
		while (j > i && a[j] >= pivot)
			--j;
		swap(&a[i], &a[j]);
		while (i < j && a[i] <= pivot)
			++i;
		swap(&a[i], &a[j]);
	}
	if (e - i + 1 == k)
		return;
	else if (e - i + 1 > k)
		FindMaxK(a, i + 1, e, k);
	else
		FindMaxK(a, s, i - 1, k - e + i - 1);
}
int main()
{
	int n;
	scanf("%d", &n);
	for (int i = 0; i < n; ++i)
		scanf("%d", &a[i]);
	int k;
	scanf("%d", &k);
	FindMaxK(a, 0, n - 1, k);
	qsort(&a[n-k], k, sizeof(int), compare);
	for (int i = n - 1; i >= n - k; --i)
		printf("%d\n", a[i]);
	return 0;
}
```
其核心函数`FindMaxK`利用`pivot`来做快速排序，将其放在最终位置上，此时`pivot`右边的元素都比它大，利用这个原理，找出`k`个元素进行排序即可

### Q5
[求排列的逆序数](http://cxsjsxmooc.openjudge.cn/2020t2springall/011/)

在Internet上的搜索引擎经常需要对信息进行比较，比如可以通过某个人对一些事物的排名来估计他（或她）对各种不同信息的兴趣，从而实现个性化的服务。

对于不同的排名结果可以用逆序来评价它们之间的差异。考虑1,2,…,n的排列i1，i2，…，in，如果其中存在j,k，满足 j < k 且 ij > ik， 那么就称(ij,ik)是这个排列的一个逆序。

一个排列含有逆序的个数称为这个排列的逆序数。例如排列 263451 含有8个逆序(2,1),(6,3),(6,4),(6,5),(6,1),(3,1),(4,1),(5,1)，因此该排列的逆序数就是8。显然，由1,2,…,n 构成的所有n!个排列中，最小的逆序数是0，对应的排列就是1,2,…,n；最大的逆序数是n(n-1)/2，对应的排列就是n,(n-1),…,2,1。逆序数越大的排列与原始排列的差异度就越大。

现给定1,2,…,n的一个排列，求它的逆序数。
- 输入

	第一行是一个整数n，表示该排列有n个数（n <= 100000)。

	第二行是n个不同的正整数，之间以空格隔开，表示该排列。
- 输出

	输出该排列的逆序数。
- 样例输入
	```markdown
	6
	2 6 3 4 5 1
	```
- 样例输出
	```markdown
	8
	```
- 提示

	1. 利用二分归并排序算法（分治）；

	2. 注意结果可能超过int的范围，需要用long long存储。

正如提示所言，先看看归并排序
```c
int a[maxsize], L[maxsize / 2 + 1], R[maxsize / 2 + 1];
void merge(int a[], int l, int ld, int rd)
{
	int i, j, k;
	int n = ld - l + 1;
	int m = rd - ld;
	for (i = 0; i < n; ++i) L[i] = a[l + i];
	for (j = 0; j < m; ++j) R[j] = a[ld + 1 + j];
	L[n] = R[m] = INF;
	for (i = 0, j = 0, k = l; k <= rd; ++k)
		if (L[i] <= R[j]) a[k] = L[i++];
		else a[k] = R[j++];
}
void mergesort(int a[], int l, int rd)
{
	if (l < rd) {
		int ld = (l + rd) / 2;
		mergesort(a, l, ld);
		mergesort(a, ld + 1, rd);
		merge(a, l, ld, rd);
	}
}
```

逆序数即`R[m]`中出现比`L[n]`更小的数时，记录逆序数的个数
```markdown
if (L[i] <= R[j]) {
			a[k] = L[i++];
		}
		else {
			a[k] = R[j++];
			cnt += n - i;
		}
```

全部代码
```c
#include<stdio.h>
#include<stdlib.h>
#define maxsize 100000
#define INF 999999
int a[maxsize], n;
long long cnt = 0;
int L[maxsize / 2 + 1], R[maxsize / 2 + 1];
void merge(int l, int ld, int rd)
{
	int i, j, k;
	int n = ld - l + 1;
	int m = rd - ld;
	for (i = 0; i < n; ++i) L[i] = a[l + i];
	for (j = 0; j < m; ++j) R[j] = a[ld + 1 + j];
	L[n] = R[m] = INF; // 哨兵
	for (i = 0, j = 0, k = l; k <= rd; ++k) {
		if (L[i] <= R[j]) {
			a[k] = L[i++];
		}
		else {
			a[k] = R[j++];
			cnt += n - i;
		}
	}
}
void mergesort(int l, int rd)
{
	if (l < rd) {
		int ld = (l + rd) / 2;
		mergesort(l, ld);
		mergesort(ld + 1, rd);
		merge(l, ld, rd);
	}
}
int main()
{
	scanf("%d", &n);
	for (int i = 0; i < n; ++i)
		scanf("%d", &a[i]);
	mergesort(0, n - 1);
	printf("%lld\n", cnt);

	return 0;
}
```