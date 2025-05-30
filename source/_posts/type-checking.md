---
title: 一场由毒蛇评论引起的检查思考
date: 2025-05-11
tag:
  - type
---

起因是在玩 [GitHub 锐评生成器](https://github.com/BingyanStudio/github-analyzer)，结果LLM评论了“至于那些在vscode-go 里补缺失的反引号的行为，建议直接给 Markdown 发明个类型检查器”。好好好，我们来讨论一下类型检查器。

![](/images/毒蛇评论.png)

<!-- more -->

那么现在的检查器（至少作者现在看来）有什么问题呢？每个编程语言的编译器写一个类型检查器，每一个编程语言的formatter（谁说formatter不是类型检查器呢，只不多是带位置数值信息的类型罢了，暴论）自己写一套，每个linter自己写一套。甚至word也有自己的类型检查器（拼写检查器）。

不说编译器的检查器，单从linter/formatter来说：

- C/C++
  - clang-tidy
  - clang-fmt
- Go
  - staticcheck
  - golangci-lint
  - gofmt
- Rust
  - clippy
  - cargo fmt
- Markdown
  - Prettier
- Typst
  - 暂时还找到

那需要一个什么类型检查器呢？

- 简洁易用（显然这是很难的）
  - 不同的语言和工具链的检查部分可以统一到这个框架中
    - 包括formatter, type checker
- 有部分规则模糊设置，这部分设置由人工配置
  - 只是因为部分事实可能人很容易知道，但是却不容易被规则所描述
  - 这里留出的模糊接口设置，除了为人工所使用之外，也可以为语言模型使用
  - 类似于证明中的假设（lean中的sorry），这些假设虽然没有被证明，但是可以让这个系统先作为一个高效的工具存在，至少人们可以预先相信一些假设，提升检查的效率。

它应该基于一个已有的如Z3等solver，但是又进行裁剪，以让更多人能够定制自己的检查器，甚至可以对word这种非结构化文档进行检测。这里定制的实现并不一定要人工实现，也可以由语言模型实现。

说到这里，又引出了一个新的宏观想法。但其实和现实中正发生的事情近乎一致。如果我们只将目光局限在确定性和概率性上，当前正在发生的如下所示：

![Verifier和Probablistic Model的融合](/images/fuse.png)

那么对于类型检查器这种verifier来说，如果可以结合一些 probablistic model 生成一些检查器办不到的事实，虽然会破坏可靠性，但是在一些可靠性要求不高的情况下，例如辅助编程场景下，找到一些原本不可能推断的类型还是不错的。这也许会提升开发效率。

> 当然以上都是暴论，没有调研分析

