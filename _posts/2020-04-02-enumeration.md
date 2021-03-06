---
title: "算法基础part1"
tags:
  - Bit Operation
  - C/C++
  - Recursion
categories:
  - Algorithm Basics
---
去年准备考研的时候看到过一个挺有意思的[故事](https://www.zhihu.com/question/344473353/answer/813920936)：
![emoji]({{site.url}}{{site.baseurl}}/images/emoji/badlaugh.png)

你会写代码了，就是会砌猪圈了，然后就觉得自己会盖人住的房子了…… 师傅问你：那房顶怎么砌？你就说：那竖着能砌，横着有啥不能滴？于是你横着砌了一块砖，发现砌不上去了，这是为啥呢……？师傅用数据结构给你搭好了横梁，于是你能在横梁上继续砌砖了。然后就觉得自己会盖楼了…… 师傅问你，那楼板怎么砌？你就说：那一层楼能砌，两层楼有啥难滴？于是你企图把砖搬到一楼的房顶然后继续砌第二层，结果楼顶塌了，这是为啥呢……？当然你可以不学数据结构也不学算法，每次都是师傅把要紧的框架搭好，你只管进入框架里埋头砌砖，也是可以的。只是年纪大了，体力不行的时候，就砌不动了，然而那时候发现砌了一辈子砖还是不会盖房子。

那么，你还小瞧代码的灵魂吗？


## 枚举
枚举是基于逐个尝试答案的一种问题求解策略，上题

### Q1
[特殊密码锁](http://cxsjsxmooc.openjudge.cn/2020t2springall/001/)

有一种特殊的二进制密码锁，由n个相连的按钮组成(n<30)，按钮有凹/凸两种状态，用手按按钮会改变其状态。然而让人头疼的是，当你按一个按钮时，跟它相邻的两个按钮状态也会反转。当然，如果你按的是最左或者最右边的按钮，该按钮只会影响到跟它相邻的一个按钮。当前密码锁状态已知，需要解决的问题是，你至少需要按多少次按钮，才能将密码锁转变为所期望的目标状态。
- 输入：
两行，给出两个由0、1组成的等长字符串，表示当前/目标密码锁状态，其中0代表凹，1代表凸。
- 输出：
至少需要进行的按按钮操作次数，如果无法实现转变，则输出impossible。
- 样例输入
```markdown
011010110101010111111011111
000010111000000001101001110
```
- 样例输出
```markdown
15
```

这一题的思路并不难，只要分清楚第一个按钮有两种情况，而后面的事情都是确定的。这是我的思路：
```c
#include <stdio.h>>
#include <string.h>
#define min(a,b) a < b ? a : b

void Flip(char* n){
	if (*n - '0')*n = '0';
	else *n = '1';
}

int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	char ori[40], tmp[40], dest[40];
	scanf("%s", ori);
	scanf("%s", dest);

	int N = strlen(ori);
	int minTimes = 9999;
	for (int p = 0; p < 2; ++p) {
		strcpy(tmp, ori);
		int times = 0;
		int curButton = p;
		for (int i = 0; i < N; ++i) {
			if (curButton) {
				++times;
				if (i > 0)
					Flip(&tmp[i - 1]);
				Flip(&tmp[i]);
				if (i < N - 1)
					Flip(&tmp[i + 1]);
			}
			if (tmp[i] != dest[i])
				curButton = 1;
			else
				curButton = 0;
		}
		if (!strcmp(tmp, dest))
			minTimes = min(minTimes, times);
	}
	if (minTimes == 9999)
		printf("impossible\n");
	else
		printf("%d", minTimes);
	return 0;
}
```
然而官方答案给了一种不同的思路——**位运算**，在学C的时候记得书上提到过位运算的效率要高得多，然而我用的极少几乎都快忘了= =
```c++
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

int oriLock;
int lock;
int destLock;

inline void SetBit(int& n, int i, int v)
{
	if (v)
		n |= (1 << i);
	else
		n &= ~(1 << i);
}
inline void FlipBit(int& n, int i)
{
	n ^= (1 << i);
}
inline int GetBit(int n, int i)
{
	return (n >> i) & 1;
}
int main()
{
	char line[40];
	destLock = lock = oriLock = 0;
	cin >> line;
	int N = strlen(line);
	for (int i = 0; i < N; ++i)
		SetBit(oriLock, i, line[i] - '0');
	cin >> line;
	for (int i = 0; line[i]; ++i)
		SetBit(destLock, i, line[i] - '0');

	int minTimes = 1 << 30;
	for (int p = 0; p < 2; ++p) {
		lock = oriLock;
		int times = 0;
		int curButton = p;
		for (int i = 0; i < N; ++i) {
			if (curButton) {
				++times;
				if (i > 0)
					FlipBit(lock, i - 1);
				FlipBit(lock, i);
				if (i < N - 1)
					FlipBit(lock, i + 1);
			}
			if (GetBit(lock, i) != GetBit(destLock, i))
				curButton = 1;
			else
				curButton = 0;
		}
		if (lock == destLock)
			minTimes = min(minTimes, times);
	}
	if (minTimes == 1 << 30)
		cout << "impossible" << endl;
	else
		cout << minTimes << endl;
	return 0;
}
```
其中有几处值得揣摩：
函数定义为`inline`即:内联函数，可以避免频繁调用函数对栈内存重复开辟所带来的消耗，但只适合简单代码的使用，如果函数体内的代码比较长，使用内联将导致内存消耗代价较高。关键字`inline`必须与函数定义体放在一起才能使函数成为内联，仅将`inline`放在函数声明前面不起任何作用。
另外，`int &n`修改n实际修改的是整数型变量自己，`int *n`修改n实际修改的是指向的整数型变量，而不是指针自己。
- 将输入的字符串转化为二进制整数，以降低空间复杂度
- 利用`curButton`作为哨兵，操作方便，结构清晰

### Q2
[拨钟问题](http://cxsjsxmooc.openjudge.cn/2020t2springall/002/)

有9个时钟，排成一个3*3的矩阵。
```markdown
|-------|    |-------|    |-------|
|       |    |       |    |   |   |
|---O   |    |---O   |    |   O   |
|       |    |       |    |       |
|-------|    |-------|    |-------|
    A            B            C

|-------|    |-------|    |-------|
|       |    |       |    |       |
|   O   |    |   O   |    |   O   |
|   |   |    |   |   |    |   |   |
|-------|    |-------|    |-------|
    D            E            F

|-------|    |-------|    |-------|
|       |    |       |    |       |
|   O   |    |   O---|    |   O   |
|   |   |    |       |    |   |   |
|-------|    |-------|    |-------|
    G            H            I
```
现在需要用最少的移动，将9个时钟的指针都拨到12点的位置。共允许有9种不同的移动。如下表所示，每个移动会将若干个时钟的指针沿顺时针方向拨动90度。
```markdown
移动    影响的时钟
 1         ABDE
 2         ABC
 3         BCEF
 4         ADG
 5         BDEFH
 6         CFI
 7         DEGH
 8         GHI
 9         EFHI
```
- 输入：
9个整数，表示各时钟指针的起始位置，相邻两个整数之间用单个空格隔开。其中，0=12点、1=3点、2=6点、3=9点。
- 输出：
输出一个最短的移动序列，使得9个时钟的指针都指向12点。按照移动的序号从小到大输出结果。相邻两个整数之间用单个空格隔开。
- 样例输入：
```markdown
3 3 0
2 2 2
2 1 2
```
- 样例输出：
```markdown
4 5 8 9
```

看到这题我的第一反应就是暴力求解，复杂度4e9大概率不会超时，代码像极了憨憨所为
```c
#include <stdio.h>>

int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	int ori[10], ways[10];//A - I == 0 - 8
	for (int i = 1; i <= 9; ++i) {
		scanf("%d", &ori[i]);
		ways[i] = i;
	}
	int p1, p2, p3, p4, p5, p6, p7, p8, p9;
	for(p1 = 0; p1 < 4;++p1)
	  for(p2 = 0; p2 < 4;++p2)
		for (p3 = 0; p3 < 4; ++p3)
		  for (p4 = 0; p4 < 4; ++p4)
			for (p5 = 0; p5 < 4; ++p5)
			  for (p6 = 0; p6 < 4; ++p6)
				for (p7 = 0; p7 < 4; ++p7)
				  for (p8 = 0; p8 < 4; ++p8)
					for (p9 = 0; p9 < 4; ++p9)
					  if ((ori[1] + p1 + p2 + p4)%4 == 0 &&
						(ori[2] + p1 + p2 + p3 + p5)%4 == 0 &&
						(ori[3] + p2 + p3 + p6)%4 == 0 &&
						(ori[4] + p1 + p4 + p5 + p7)%4 == 0 &&
						(ori[5] + p1 + p3 + p5 + p7 + p9)%4 == 0 &&
						(ori[6] + p3 + p5 + p6 + p9)%4 == 0&&
						(ori[7] + p4 + p7 + p8)%4 == 0&&
						(ori[8] + p5 + p7 + p8 + p9)%4 == 0 &&
						(ori[9] + p6 + p8 + p9)%4 == 0) {
						ways[1] = p1;
						ways[2] = p2;
						ways[3] = p3;
						ways[4] = p4;
						ways[5] = p5;
						ways[6] = p6;
						ways[7] = p7;
						ways[8] = p8;
						ways[9] = p9;
					  }
	for (int i = 1; i < 10; ++i) {
		if (ways[i] != 0) {
			for (int j = ways[i]; j > 0; --j)
				printf("%d ", i);
		}
	}

	return 0;
}
```
结果10ms通过，还行吧，因为官方答案居然耗时了200ms，代码如下
```c++
#include <iostream>
#include <bitset>
#include <algorithm>
#include <functional>
#include <cstring>
using namespace std;
int oriClocks[9];
int clocks[9];
const char * moves[9] = { "ABDE","ABC","BCEF","ADG","BDEFH","CFI","DEGH","GHI","EFHI" };
int moveTimes[9] = {0};
int result[9];
int minTimes = 1 << 30;
void Enum(int n)
{
	if(n >= 9) {
		memcpy(clocks,oriClocks,sizeof(clocks));
		int totalTimes = 0;
		for(int i = 0; i < 9; ++i) {
			if(moveTimes[i]) {
				for(int k = 0; moves[i][k]; ++k) {
					clocks[moves[i][k]-'A'] = (clocks[moves[i][k]-'A'] + moveTimes[i]) % 4;
					totalTimes += moveTimes[i];
				}
			}
		}
		int i;
		for(i = 0; i < 9; ++i)
			if(clocks[i])
				break;
		if(i == 9) {
			if( minTimes > totalTimes) {
				minTimes = totalTimes;
				memcpy(result,moveTimes,sizeof(result));
			}
		}
		return ;
	}
	for(int i = 0; i < 4; ++i) {
		moveTimes[n] = i;
		Enum(n+1);
	}
	return ;
}
int main()
{
	for(int i = 0; i < 9 ; ++i)
		cin >> oriClocks[i];
	Enum(0);
	for(int i = 0; i < 9; ++i)
		for(int k = 0; k < result[i]; ++k)
			cout << i+1 << " ";
	return 0;
}
```
这段代码有个点不容易理解
```c++
	for(int i = 0; i < 4; ++i) {
		moveTimes[n] = i;
		Enum(n+1);
	}
```
这段代码的意思其实是用递归的方式列举4e9种可能，因为递归调用栈，导致复杂度比9层循环还要大
- 用递归的方式可以增加代码的可读性，但是牺牲了时间空间