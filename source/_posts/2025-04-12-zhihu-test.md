---
title: 知乎markdown导入测试
date: 2025-04-12
mathjax: true
---

测试知乎markdown导入。[VSCode-Zhihu](https://github.com/niudai/VSCode-Zhihu)不可用了。

<!-- more -->

[源码链接](https://github.com/Lslightly/Lslightly.github.io/blob/bfe45667adbdd03541641e0d70913a800ab52767/source/_posts/2025-04-12-zhihu-test.md).


# 一级标题

## 二级标题

**加粗**, _斜体_

> 哦，这就是markdown导入对列表的处理吗😂

1. 1
   1. 1.1
      1. 1.1.1
   2. 1.2
2. 2
3. 

- i1
- i2
  - i2.2
  - i2.3
  - i2.4
  - i4
- 

> quote


> quote
> 
> the last line is empty

---

```go
func main() {
   fmt.Println("Hello, World")
}
```

markdown link:

[1], [baidu](https://www.baidu.com)

annotation [^1]

image

{% img /images/apple-touch-icon.png %}

math: inline $e^{\pi i} + 1 = 0$

$$
e = mc^2
$$

| head 1 | head 2 |
| ------ | ------ |
| abc    | abd    |
| cab    | ...    |

- [ ] 合并单元格

[1]: https://www.baidu.com
[^1]: annotation
