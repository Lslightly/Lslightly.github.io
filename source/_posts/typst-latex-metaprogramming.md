---
title: 从typst和latex比较引发的metaprogramming
tags:
    - metaprogramming
---

起源是我在知乎上[关于typst let和latex newcommand的区别提问](https://www.zhihu.com/question/638846250)。其实这背后也隐藏着metaprogramming的影子。

<!-- more -->

心血来潮想回答一下这个一年前提的问题。

> 一年过去了怎么也没有人回答呢？

搜了一下reddit没有搜到相关的信息。

那我们先问问kimi是怎么看待的吧？以下内容部分结合了[share link](https://kimi.moonshot.cn/share/d0g1rsjacc45tdd2coe0)。

Question: typst的let和latex的newcommand之间的区别，以及衍生的metaprogramming的相关知识。已联网，长思考。

基本功能和语法：

- Typst 的 `let` ：用于定义变量、函数等。例如定义变量 `let a = 1` ，定义简单函数 `let add = (a, b) => {a + b}` ，定义带有多行代码块的函数 `let complexFunc = (arg) => { if arg > 0 { arg + 1 } else { arg - 1 }}`
- LaTeX 的 `\newcommand` ：用于定义新的命令。例如定义不带参数的命令 `\newcommand{\R}{\mathbb{R}}` ，定义带一个参数的命令 `\newcommand{\vektor}[1]{\mathbf{#1}}` ，定义带可选参数的命令 `\newcommand{\power}[2][2]{#1^{#2}}`

那很显然，从编程的角度来说，let可以自行定义带语义信息的变量名。而newcommand只能使用 #1, #2等位置参数，这不够直观。

作用域：

`let` 有 [block](https://typst.app/docs/reference/scripting#blocks) 的概念，例如 `{ let x = 1; x + 2 }`，防止定义冲突。

灵活性：

`let`提供函数式编程能力，可以创建高阶函数。

例如如下代码构造了一个高阶函数 `adda`，并在 `add` 函数中调用了这个函数来生成 `adda(a)` 函数。见 [typst.app project](https://typst.app/project/rv3QI82XHXSbZeK248eGYE)。

```typst
#let adda(a) = (b) => a+b
#let add(a, b) = {
  return adda(a)(b)
}

#add(1, 2)
```

latex中的元编程。latex也提供了if, else, forloop等功能。

[macroprogramming现状论文](https://dl.acm.org/doi/pdf/10.1145/3579353)。但是这东西看survey是看不出东西的，本来就是编程的东西。



