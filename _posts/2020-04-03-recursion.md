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
测试1ms通过，算是完美达成目标。然后来看下官方代码
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
官方答案有点像选择排序，从小到大从`str`中选出不同的元素，选满了n个就输出，测试发现跑了6ms，还行咯。

### Q2
[2的幂次方表示](http://cxsjsxmooc.openjudge.cn/2020t2springall/004/)

任何一个正整数都可以用2的幂次方表示。例如：

    137=2^7+2^3+2^0

同时约定方次用括号来表示，即ab可表示为a(b)。由此可知，137可表示为：

    2(7)+2(3)+2(0)

进一步：7=22+2+20（21用2表示）

    3=2+2^0

所以最后137可表示为：

    2(2(2)+2+2(0))+2(2+2(0))+2(0)

又如：

    1315=2^10+2^8+2^5+2+1

所以1315最后可表示为：

    2(2(2+2(0))+2)+2(2(2+2(0)))+2(2(2)+2(0))+2+2(0)

- 输入：
一个正整数n（n≤20000）。
- 输出：
一行，符合约定的n的0，2表示（在表示中不能有空格）。
- 样例输入
```markdown
137
```
- 样例输出
```markdown
2(2(2)+2+2(0))+2(2+2(0))+2(0)
```

这道题就是典型的递归，最小子项为`2`和`2(0)`，当n是2的m次幂时，`2(power(m))`递归计算；否则分为两项递归，其中一项为2的m次幂。代码如下：

```c
#include <stdio.h>

void Power(int n)
{
	if (n == 1)printf("2(0)");
	else if (n == 2)printf("2");
	else {
		int a = 1, b, cnt = -1;
		while (a <= n) {
			a *= 2;
			++cnt;
		}
		a /= 2;
		b = n - a;
		if (a > 2) {
			printf("2(");
			Power(cnt);
			printf(")");
		}
		else printf("2");
		if (b != 0) {
			printf("+");
			Power(b);
		}
	}
}

int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	int n;
	scanf("%d", &n);
	Power(n);

	return 0;
}
```
再来看官方代码

```c++
#include <iostream>
using namespace std;

inline int GetBit(int n, int i)
{
	return (n >> i) & 1;
}
void Print(int n) {
	bool first = true;
	for(int i = 15; i >= 0; --i) {
		if(GetBit(n, i)) {
			if(!first) {
				cout << "+";
			}
			else
				first = false;
			if(i == 0)
				cout << "2(0)";
			else if(i == 1)
			 	cout << "2";
			else {
				cout << "2(";
				Print(i);
				cout << ")";
			}
		}
	}
}
int main()
{
	int n;
	cin >> n;
	Print(n);

	return 0;
}
```
- 使用位运算来计算`<=n`的最大的2的`i`次幂

### Q3
[Boolean Expressions](http://cxsjsxmooc.openjudge.cn/2020t2springall/005/)

The objective of the program you are going to produce is to evaluate boolean expressions as the one shown next:

Expression: `( V | V ) & F & ( F | V )`

where V is for True, and F is for False. The expressions may include the following operators: `!`for not, `&`for and, `|`for or, the use of parenthesis for operations grouping is also allowed.

To perform the evaluation of an expression, it will be considered the priority of the operators, the not having the highest, and the or the lowest. The program must yield V or F , as the result for each expression in the input file.
- 输入：
The expressions are of a variable length, although will never exceed 100 symbols. Symbols may be separated by any number of spaces or no spaces at all, therefore, the total length of an expression, as a number of characters, is unknown.
The number of expressions in the input file is variable and will never be greater than 20. Each expression is presented in a new line, as shown below.
- 输出：
For each test expression, print "Expression " followed by its sequence number, ": ", and the resulting value of the corresponding test expression. Separate the output for consecutive test expressions with a new line.
Use the same format as that shown in the sample output shown below.
- 样例输入
```markdown
( V | V ) & F & ( F| V)
!V | V & V & !F & (F | V ) & (!F | F | !V & V)
(F&F|V|!V&!F&!(F|F&V))
```
- 样例输出
```markdown
Expression 1: F
Expression 2: V
Expression 3: V
```

- 表达式递归求值的典型套路

```c++

#include<iostream>
#include<cctype>
using namespace std;

int expression();//表达式
int term();//项
int factor();//因子

int main()
{
	cout << expression() <<endl;
	return 0;
}

int expression()
{
	int result = term();
	char c = cin.peek();
	while(c == '+' || c == '-')
	{
		cin.get();
		if(c == '+')
		result += term();
		else if(c == '-')
		result -= term();
		c = cin.peek();
	}
	return result;
}

int term()
{
	int result = factor();
	char c = cin.peek();
	while(c == '*' || c == '/')
	{
		cin.get();
		if(c == '*')
		result *= factor();
		if(c == '/')
		result /= factor();
		c = cin.peek();
	}
	return result;
}

int factor()
{
	int result = 0;
	char c = cin.peek();
	if(c == '(')
	{
		cin.get();
		result += expression();
		cin.get();
	}
	else while(isdigit(c))
	{
		cin.get();
		result = result * 10 + c - '0';
		c = cin.peek();
	}

	return result;
}
```
这里主要是符号的不同，由原来的`+``-``*``/`变成了`&``|``!`，其中优先级依次是`! > & > |`，按本题的要求更换一些参数就OK了

```c
#include <stdio.h>
#include <string.h>

int expression();
int term();
int factor();
char p[110];
int f = 0;
int expression()
{
	int result = term();
	while (1) {
		char op = p[f]; //读入一个字符
		if (op == '&' || op == '|') {
			f++; //取走一个字符
			int value = term();
			if (op == '&') result = result && value;
			else result = result || value;
		}
		else
			break;
	}
	return result;
}
int term()
{
	int result;
	char op = p[f];
	if (op == '!') {
		f++;
		result = !factor();
	}
	else
		result = factor();

	return result;
}
int factor()
{
	int result = 0;
	char c = p[f];
	if (c == '(') {
		f++;
		result = expression();
		f++;
	}
	else if (c == 'V') {
		result = 1;
		f++;
	}
	else if (c == 'F') {
		result = 0;
		f++;
	}
	else if (c == '!') {
		result = term();
	}

	return result;
}
int main()
{
	//freopen("E:\\IDMdowanload\\in.txt", "r", stdin);
	int cnt = 0;
	char tmp[10000], c;
	while (scanf("%[^\n]", tmp) != EOF) {
		getchar();//读入`\n`
		int i = 0, j = 0, n;
		n = strlen(tmp);
		while (i < n) {
			if (tmp[i] != ' ') p[j++] = tmp[i];
			i++;
		}
		p[j] = '\0';
		printf("Expression %d: ", ++cnt);
		printf("%c\n", (expression() ? 'V' : 'F'));
		memset(tmp, 0, sizeof(tmp));
		f = 0;
	}

	return 0;
}
```
- 有以下几点需要注意

	连续读取多个带空格的字符串，[scanf](https://gadada.github.io/notes/2020-04-06-String/)的相关细节

	`!`由于优先级较高，即可以位于因子的首部，也可以位于项的首部

官方答案

```c++
#include <iostream>
#include <cstdio>
using namespace std;
char wholeExp[200];
bool exp();
bool factor();
bool item();
int ptr = 0;
bool exp() {
	bool result = item();
	while(wholeExp[ptr] == '|' ) {
		++ptr;
		result = result | item();
	}
	return result;
}
bool item() {
	bool result = factor();
	while(wholeExp[ptr] == '&') {
		++ptr;
		result = result & factor();
	}
	return result;
}
bool factor() {
	bool result;
	switch( wholeExp[ptr]) {
		case 'F':
			++ptr;
			return false;
			break;
		case 'V':
			++ptr;
			return true;
			break;
		case '(':
			++ptr;
			result = exp();
			++ptr;
			return result;
			break;
		case '!':
			++ptr;
			result = !factor();
			return result;
			break;
	}
}
int main()
{
	char c;
	int i = 0;
	int t = 1;
	int n = EOF + 1;
	while(n != EOF) {
		 n =  scanf("%c",&c);
		if(	n == EOF || c == '\n') {
			wholeExp[i] = 0;
			if( i > 0) {
				ptr = 0;
				if (exp())
					printf("Expression %d: V\n",t++);
				else
					printf("Expression %d: F\n",t++);
			}
			i = 0;
		}
		else if( c != ' ')
			wholeExp[i++] = c;
	}
}

```