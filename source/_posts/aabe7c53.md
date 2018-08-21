---
title: 谈谈递归和迭代
date: '2017/07/10 13:34:28'
updated: '2017/12/24 20:16:33'
type: categories
categories: 文字阁
tags:
  - SICP
	- 递归
	- 迭代
copyright: true
abbrlink: aabe7c53
mathjax: true
thumbnail: https://res.cloudinary.com/wincer/image/upload/v1530860277/blog/recursion_and_iteration/cover.png
---

## 前言

首次接触递归（recursion）这个概念是在学习 C 语言的时候，当时老师是根据「汉诺塔」[^1]这一具体问题的求解来介绍递归这个概念，至于迭代（iterate），好像 C 语言老师压根没提这个概念，第一次是在 MIT 的 Python 导论中听说的，但当时听完之后也只是对迭代和递归只有极其有限的了解。正好借着 SICP，好好弄清楚二者的概念。

<!-- more -->

首先明确二者的概念：

- 递归：是指在函数的定义中使用函数自身的方法。

- 迭代：迭代是程序中对一组指令（或一定步骤）的重复。在每次执行这组指令（或这些步骤）时，都从变量的原值推出它的一个新值。
循环，则是一种更宽泛的概念，凡是重复执行的一段代码，都可以称之循环，它是作为一种控制流程存在的，它与递归和迭代不同，个人认为递归和迭代更像是一种算法思想，而循环结构则是这种思想在程序中的一种体现方式。

[^1]: https://zh.wikipedia.org/wiki/%E6%B1%89%E8%AF%BA%E5%A1%94

## 递归

用一个具体的题目来说明一下。

定义一个阶乘函数：
$$n!=n\times(n-1)\times(n-2)\times\cdots\times3\times2\times1$$
有一种很容易想到的计算方式就是：对于一个正整数 $n$，$n!$ 就等于 $n$ 乘以 $(n-1)!$:
$$n!=n\times[(n-1)\times(n-2)\times\cdots\times3\times2\times1]=n\times(n-1)!$$
这样我们就能通过算出 $(n-1)!$，并将其结果乘以 $n$ 的方式来计算出 $n!$。再注意到 $1!$ 就是 $1$，我们就可以出编写一个程序：

```lisp
(define (factorial n)
  (if (= n 1)
      1
      (* n (factorial (- n 1)))))
```

我们采取代换模型分析以下当这个程序在计算 $6!$ 时的具体过程。

```lisp
(factorial 6)
(* 6 (factorial 5))
(* 6 (* 5 (factorial 4)))
(* 6 (* 5 (* 4 (factorial 3))))
(* 6 (* 5 (* 4 (* 3 (factorial 2)))))
(* 6 (* 5 (* 4 (* 3 (* 2 (factorial 1))))))
(* 6 (* 5 (* 4 (* 3 (* 2 1)))))
(* 6 (* 5 (* 4 (* 3 2))))
(* 6 (* 5 (* 4 6)))
(* 6 (* 5 24))
(* 6 120)
720
```

递归的代换模型显示的是一种逐步展开而后收缩的形状。在展开的阶段中，这一计算过程构造了一个「推迟进行」的操作所形成的链条，收缩阶段表现为这些运算的实际执行。这种类型的计算过程由一个「推迟进行」的运算链条刻画，称之为「递归计算过程」。

要执行这种计算过程，解释器（或者编译器）就需要保护好那些以后将要执行的操作的轨迹。

在计算 $n!$ 时，推迟执行的乘法链条的长度就是为保存其轨迹需要保存的信息量，这个长度随着 $n$ 值而线性增长，就像计算中的步骤数目一样。这样的计算过程成为**线性递归过程**。

## 迭代

我们同样可以将计算阶乘$n!$的规则描述为：先乘 $1$ 和 $2$，而后将结果乘以 $3$，而后再乘以 $4$，这样下去直到到达 $n$。更程序化的说，我们要保存一个变动的乘积 `product`，以及一个从 $1$ 到 $n$ 的计数器 `counter`，这一计算过程可以描述为 `counter` 和 `product` 的如下变化，每一步都按照如下变化
​     $$product \longleftarrow counter\times product$$
​     $$counter \longleftarrow counter+1$$
可以看到，$n!$ 也就是当 `counter` 超过 $n$ 的时候成绩 `product` 的值。

将上述描述编写成程序：

```lisp
(define (factorial n)
  (fact-iter 1 1 n))
(define (fact-iter product counter max-count)
  (if (> counter max-count)
      product
      (fact-iter (* counter product)
                 (+ counter 1)
                 max-count)))
```

与递归类似，同样采取代换模型来查看 $6!$ 详细过程：

```lisp
(factorial 6)
(factorial 1 1 6)
(factorial 1 2 6)
(factorial 2 3 6)
(factorial 6 4 6)
(factorial 24 5 6)
(factorial 120 6 6)
(factorial 720 7 6)
```

迭代的计算过程并没有任何增长或者收缩。对于任意一个 $n$，在计算过程中的每一步，我们所需要保存的就只有变量 `product`、`counter`、`max-count` 的**当前值**。这种过程就称为「迭代计算过程」。

「迭代计算过程」就是状态可以用固定数目的变量来描述的计算过程，但不仅仅如此，还需要一套规则来描述计算过程从这个状态到下一状态转换时，这些变量的更新方式以及结束时的检测。

在计算 $n!$ 的时候，所需计算步骤随着 $n$ 线性增长，这种过程成为**线性迭代过程**。

## 结语

在做迭代和递归之间比较的时候，我们需要注意「递归计算过程」和「递归过程」的概念。当我们说一个程序是递归的时候，论述的是一个程序在语法层面上的一种形式：（直接或者间接地）引用了该程序的本身。在说某一计算过程具有某种模式时，我们说的是这一计算过程的进展方式（如，先伸展后收缩等等），而不是书写相应程序所体现的语法形式。

我们可以说 `fact-iter` 是一个递归过程（调用了自身），但是 `fact-iter` 产生的是一个迭代的计算过程，这么说可能会感觉很不舒服，甚至第一次听还会觉得这句话是错误的。但是这一计算过程确实是迭代的，因为它的状态由三个变量刻画就足够了。

**本文系本人阅读 SICP 时所做的笔记，全部笔记可在标签 SICP 中查看**