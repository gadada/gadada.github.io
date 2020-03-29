---
title: "G(f(n-1), f(n-2), ... f(n-m))的尾递归实现"
categories:
  - Functional Programming
tags:
  - Tail recursive
  - WIP
---

尾递归是函数式编程中很重要的一个概念，在面向对象或者面向过程中的循环在函数式编程模式中会用到尾递归来实现。

 > A call is said to be in tail position if the caller does nothing other than return the value of the recursive call.

如果一个函数中，所有的递归调用都是尾递归调用的话，许多语言都能够将这类调用在编译阶段转换为循环，从而避免一般递归调用可能会出现的`StackOverflow`的问题。所以，我们来探究一下，到底如何来实现某些递归函数的尾递归实现方法。

## f(n) = n * f(n-1)

举一个比较经典的例子，计算阶乘。
比较直观的实现阶乘的方法是使用循环：

``` java
int factorial(int n) {
  int result = 1;
  for (int i = 1; i <= n; i++) {
    result = i * result;
  }
  result;
}
```

对于程序员来说，用循环来实现是非常直观的一种实现方式，基本上就是根据阶乘的定义出发，从小到大一步一步计算到n。另外一种从定义出发并且比较直观的方式是从`n`一步一步计算到1的方法，即，使用递归来实现：

``` scala
def factorial(n: Int): Int = {
  if (n == 1) 1
  else n * factorial(n-1)
}
```

但这种递归存在一个问题是，递归调用不是递归函数中的最后一步，（还需要再乘以n）也就是非尾递归的。一般情况下，递归调用过程中是需要被压栈和出栈的，因此如果递归调用深度很深的话，很容易造成调用栈溢出的问题。但尾递归作为递归中的一种特例，它是可以被优化的，编译器在编译过程中很容易将尾递归变成循环，从而避免调用栈溢出的问题。

一般情况下，如果想要使用递归方式来实现某个函数的话，最好使用尾递归来实现。例如，刚才的例子中，我们可以通过尾递归来实现：

``` scala
def factorial(n: Int): Int = {
  def loop(n: Int, acc: Int): Int = {
    if (n == 1) acc
    else loop(n-1, n * acc)
  }

  loop(n-1, 1)
}
```


## f(n) = f(n-1) + f(n-2)

再来看一个例子，求斐波那契数列。同样，求Fibnacci数列的最直观的方法就是使用它的定义，直接上递归：

``` scala
def fibnacci(n: Int): Int = {
  if (n == 1) 0
  else if (n == 2) 1
  else {
    fibnacci(n-1) + fibnacci(n-2)
  }
}
```

同样，这也是一种非尾递归，存在调用栈溢出的风险。

那好，换一种尾递归解法：

``` scala
def fibnacci(n: Int): Int = {
  def loop(n: Int, fibnminus1: Int, fibnminus2: Int): Int = {
    if (n == 1) fibnminus2
    else if (n == 2) fibnminus1
    else loop(n-1, fibnminus1+fibnminus2, fibnminus1)
  }

  loop(n, 1, 0)
}
```

实际上，上面这个解法不是我自己想出来的，我想了半天没想出来，于是在网上搜了一下，发现别人给的这个解法是这样的。
然后我就想，是否存在递归的通用尾递归实现方法呢？所以才有了这篇文章。

## 从前两个例子想到的

在求阶乘和Fibnacci数列过程中，我们发现`loop`函数中，除了需要传递`n`以外还需要传递额外的参数。
对于阶乘，需要额外的一个参数，对于Fibnacci数列，则需要两个。
这个额外参数的个数刚好和递归定义中，后一个值`f(n)`所依赖的前值`f(n-1)`或者`f(n-1)+f(n-2)`中的个数是一致的。
细想一下，其实这些前值都是对递归运算过程中的状态的保存，而后一个值所依赖的状态刚好就是这些计算结果。
那么是不是可以得到结论，对于递归定义`f(n) = G(f(n-1), f(n-2), ... f(n-m))`，其中`G`表示接受m个参数的函数，
是不是只要保存m个状态就可以用尾递归来实现了呢？

## f(n) = G(f(n-1), f(n-2), ... f(n-m))

我们假设`f(n)`可以用刚才提到的这个思路来解决，那么尾递归实现代码应该是这样的：

``` scala
def f(n: Int): Int = {
  def G(states: Array[Int]): Int = {
    // G in f(n) definition
    // f(n-i) is in states[i]
  }

  def loop(n: Int, states: Array[Int]): Int = {
    if (n < m) states(n-1)
    else {
      // 保存新的状态，即，淘汰最老的，加入最新的
      val newStates = states.slice(1, m) ++ Array(G(states))
      loop(n-1, newStates)
    }
  }

  val states = Array(f(1), ... f(m-1))
  loop(n, states)
}
```

例如，在阶乘计算过程中，`G(x) = n * x`，而在Fibnacci数列计算过程中`G(x1, x2) = x1 + x2`。
接下来，我们就要验证并证明该`loop`方法计算所得的`f(n)`是否和`f(n) = G(f(n-1), f(n-2), ..., f(n-m))`计算所得的相等。

### 证明

我们通过归纳法来证明本算法的正确性。

