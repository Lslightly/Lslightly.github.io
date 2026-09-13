---
title: 软件工程实践方法论
date: 2026-09-13
tags: SE, agent
---

现在在做大型的rust项目(servo)。一个使用agent比较困难的点在于，如何控制agent的上下文范围，以及是否需要给agent设置上下文范围。

对于这种大型项目，如果没有提前的一个overview架构分析的话，人本身不是很好上手，agent也会陷入和人一样的需要各种查找的情况。这时候就不一定想的清楚。

目前就用了codegraph和rust-analyzer lsp，但感觉还是不够高效。

应该找一个人可以看懂，同时又方便agent理解阅读的可操作中介。但是我不知道有什么。

## 现在的调试做法（似乎比较低效）

按照某个例子以及打日志的方式，看整体流程。有web内核整体的工作思路，没有JS对象的模型，也没有web内部API操作了什么对象的概览。web内核 DOM API实在太多了。

我是先看打日志，同时在调日志的过程中看流程。但是我没有梳理UML类图，所以存在遗忘的情况。整个流程非常复杂，UML类图、流程图、时序图感觉还不够。

我感觉我还是没有掌握快速入门一个项目的精髓，效率还是有点低效。如果有什么好的开发者心得，是不是更好一些？但是项目不一样，所使用的方法可能也不同。

## 值得尝试的方向以及相应的工具

整理下来，判断一个"中介"值不值得做的标准其实只有一条：**要么它能从源码重新生成，要么它被一个能跑起来的断言钉住。** 手工维护的文档和手画的图，在 servo 这种规模的仓库里大概活不过两周。按这条标准，想试的方向分三层。

### 一、自动生成：不腐烂的索引

这类东西不是给我读的，是给我和 agent 查的。重点是查询能力，不是产物本身。

- **WebIDL → 接口继承图**。`components/script/dom/*.webidl` 里的 `interface X : Y` 就是类层级本身，写个脚本渲成图就行，每次重新跑一遍就不会腐烂。这直接回答了上面"没有 JS 对象模型"的问题——模型一直都有，只是没人把它抽出来。
- **规范自带的索引**。HTML/DOM spec 自带 IDL index，以及事件循环、渲染的图。标准委员会在替我们维护，不要自己重画。
- **代码结构**。`cargo modules` / `cargo depgraph` 出模块依赖图，rust-analyzer 的 crate graph 和 call hierarchy 做按需下钻。

### 二、可运行：轨迹配方

把"先看日志再看流程"固化成一份能直接执行的配方：一个固定的 WPT 用例 + 一条确切的命令 + 日志过滤前缀 + 期望看到什么 + 看到 A 说明走了哪条路。它的价值在于**是命令而不是描述**——agent 能直接跑，我能直接复现，也不会因为流程变了就失效。

配套想试的是 **rr** 这类 record-replay 工具。servo 是并发加多进程 IPC，正序打日志线性又不可逆，只能靠记住前面看过什么。rr 能倒着走、能反复回到同一时刻，相当于用反向调试替代记忆——正好治"没有梳理类图所以存在遗忘"这个毛病。

### 三、被测试钉住

任何一条搞明白了的不变量，立刻写成断言，或者一个小 WPT 用例。这是唯一同时能防我遗忘、又防 agent 踩坑的机制：我忘了没事，测试不会忘。

### 四、两条配套的方法

**先固定一条垂直切片。** 挑一个用例当标本，之后学到的东西都挂在这条切片上。不要横向铺开去理解架构——横向铺是遗忘最快的路径。架构应该是穿过一次具体修改的副产品，不是动手的前提。

**上下文范围不按文件划，按任务契约划。** 按文件划，agent 看不见跨层后果，容易做出局部正确全局错误的事；完全不划，它会读两百个文件然后开始模式匹配。给四件套：目标 + 验收方式（一个跑得起来的测试）+ 入口点 + 禁区。地图和检索能力给全，文件不给全，同时要求每个结论都带 `file:line`。

## 具体的工具与插件

按上面的三个层次，把现成的、能直接拿来用的东西列一下。有些是给 agent 用的，有些纯粹是给自己用的。

### 给 agent 接上语义层

痛点是 agent 默认只能 grep，而 grep 在 servo 这种规模下等于瞎猜。

- **[mcp-language-server](https://github.com/isaacphi/mcp-language-server)**。把它架在 rust-analyzer 前面，`definition` / `references` / `hover` / `diagnostics` 就变成了 agent 能调的 MCP 工具。这个改动成本最低，因为 rust-analyzer 我本来就在用，只是 agent 现在够不着它。
- **[Serena](https://github.com/oraios/serena)**。在 LSP 之上封装成符号级工具：`get_symbols_overview`、`find_symbol`、`find_referencing_symbols`。它把"按符号检索"而不是"按文件读"变成了 agent 的默认动作，直接对应上面说的"上下文范围按任务契约划"。它还有个 `.serena/memories/` 做跨会话的项目记忆，本质上就是我在找的那种中介。
- **rust-analyzer 自己的 SCIP 索引**。`rust-analyzer scip .` 能吐出一个静态索引文件（protobuf），全仓库的定义、引用、实现都在里面。好处是**离线**：agent 查一次索引，比每次现场做符号解析便宜得多。

### 给 agent 一张地图

- **`cargo metadata`**。整个 crate 和依赖图是 JSON，直接喂给 agent，比让它一个个读 `Cargo.toml` 强太多。这条几乎零成本，我居然一直没做。
- **`cargo modules` / `cargo depgraph`**。前者出模块树，后者出 dot 格式的依赖图。
- **[Searchfox](https://searchfox.org/)**。这个我觉得是最接近我想要的"可操作中介"的东西——它就是为 web 引擎代码做的：全文搜索 + 符号 + 交叉引用 + git blame + 测试覆盖，能直接回答"这个 WebIDL 接口谁实现了、谁调用了它"。**要注意**：Searchfox 索引的是 mozilla-central 里的那份 servo 代码（路径长得像 `servo/components/...`），不是 servo 上游仓库的最新状态，当参考模型用可以，别当权威。顺带一提，Gecko 那套 IDL ↔ 实现的交叉引用结构本身就是很好的样板。

### 让流程可回放

这条让我意外：**servo 是内置支持 rr 的**。

- `./mach run --debugger=rr testcase.html` —— 直接拿 rr 当调试器。
- `./mach test-wpt --chaos <test>` —— 用 rr 录 trace（需要 `rr` 在 PATH 里）。
- 跑单个用例：`./mach test-wpt tests/wpt/tests/dom/historical.html`。注意要用 release 构建再配 `-r` 跑，debug 构建跑 WPT 容易超时。

这就是"轨迹配方"的落地形态：一个固定用例加一条 mach 命令，可以写进文档里，也可以写成一个 skill 让 agent 按名字调。

一个环境上的前提：rr 需要 Linux，而且要能拿到 perf 事件和相应的内核设置。我本地是 Windows，如果开发环境放在远端 Linux 上就正好。

### 时序图不用画，让它生成

这条直接回应"时序图还是不够"：**把打点从 `println` 换成 span，时序图就是产物而不是手稿。**

- 用 `tracing` 的 span 按层打点，导出成 chrome trace 或 flamegraph，丢进 Perfetto / `chrome://tracing` 看。跨层的时间关系一眼可见，而且每次重跑自动更新，不存在腐烂问题。
- servo 自己已经有日志和 trace 设施，值得先看看现有的能不能直接导成 trace 格式，能不自己造就不自己造。
- **所有图都用文本格式**（mermaid / graphviz dot / D2）。理由很实际：agent 能读、能改、能 diff，PNG 只能给人看。

### 给 agent 本身配的东西

- **`CLAUDE.md` / `AGENTS.md`**：最小的中介。放构建命令、跑单个测试的命令、分层说明、禁改清单。半小时的事。
- **项目级 skill**（`.claude/skills/`）：把轨迹配方写成一个 skill，agent 按名字调用，而不是每次重新摸索一遍怎么跑。
- **hooks**：编辑后自动跑 `cargo check` / `clippy`，把反馈回路从"我想起来才跑"变成"自动跑"。
- **受限子 agent**：给一个只许搜索和读取、且只返回 `file:line` 的"导航 agent"，主 agent 的上下文就不会被文件内容淹没。这是划上下文范围最直接的手段。

### 一些传统工具

- **`cargo expand`**。servo 有大量宏和代码生成（bindings 就是生成的），展开之后才是真正的代码。
- **`ast-grep`**。结构化搜索，"找出所有实现了 X 的 impl"这类问题，文本搜索答不上来。
- **`git log -L`**。追一个函数的演变史。入门一个子系统的时候，这比读文档有用得多。
- **`scc` / `tokei`**。先知道肉长在哪。
- **WPT 自己的 `interfaces/` 目录和 idlharness 测试**。IDL 和对象模型是规范产出的，直接用，不用自己整理。

## 先做哪个

不用全都上。按成本排，前三件事大概一两天：

1. `cargo metadata` + `mcp-language-server` —— 半天，让 agent 至少能按符号而不是按文件看代码。
2. 写 `CLAUDE.md` —— 一小时，把构建、跑单个 WPT、分层、禁改清单写死。
3. 一条固定的 mach 轨迹配方 —— 挑一个 DOM 用例，把命令和期望输出记下来，之后所有理解都挂上去。

rr 和 span 时序图收益更大，但要先踩环境的坑，放在后面。

## 编译时长与内存问题

还有一个很现实的问题：因为要改一些相对上游的库，每次 servo 编译要 3 分钟，而且要 32G 内存（16G 不够，IDE 的 rust-analyzer 还有额外的开销）。这件事值得单独想一下。

### 先量，再改

不量就调参数是瞎调。三个命令：

- `cargo build --timings` —— 出一份 HTML，每个 crate 的编译时间和并行度都在里面。
- `/usr/bin/time -v ./mach build` —— 看峰值 RSS。
- `cargo llvm-lines` —— 找泛型实例化爆炸的元凶。

我的猜测是**峰值内存的主犯不是"上游库"本身，而是最终链接和少数巨型 crate 的 codegen**。链接那一步要把整个程序的符号表装进内存，往往是整个构建的峰值时刻；而 servo 这种体量，debug info 也能占掉一大块。先确认是哪一步在吃内存，再决定动哪一刀。

### 最值钱的一条：把回路拆成快慢两条

3 分钟加 32G 的本质问题是**每一次改动都要付全款**，但绝大多数改动其实不需要跑完整 servo。

- **`./mach check`**。只做类型检查，跳过代码生成。这是 servo 官方文档自己推荐的迭代方式：改了一个 crate 导致别的 crate 报错时，用它而不是全量 build。
- **`./mach build --dev -p servo`**。只编 servo 这一个包，不编 libsimpleservo。
- **最小复现 crate**。既然要改的是上游库，就 `cargo new --lib`，只依赖那个库，写一个最小重现。改-试循环从 3 分钟降到几秒，改好了再搬回 servo。**这条收益最大**，因为它把问题从"优化编译系统"变成了"组织工程"。
- 只有必须看真实渲染/行为时才走慢回路，而且**攒一批改动一起验证**，不要每改一行就全量编一次。

### 降内存峰值：不写代码就能拿到的

- **`-j` 调小**。峰值内存大致等于并行度乘以单 crate 峰值，16G 撑不住通常就是并行度太高。牺牲墙上时间换可行性，这是最直接的一招。
- **关掉 debug info**。这条我很怀疑是你 32G 的主要来源之一。cargo 支持用环境变量覆盖 profile，不用改 servo 的 `Cargo.toml`：`CARGO_PROFILE_DEV_DEBUG=false`，折中一点用 `1` 或 `line-tables-only`。注意这跟上一条建议（用 rr/gdb 回放）是打架的——没有 debug info 就没有像样的栈。所以要么保留 `line-tables-only` 这个折中，要么只在跑构建时关、调试时开。
- **关掉 LTO**：`CARGO_PROFILE_RELEASE_LTO=false`。fat LTO 要把整个程序的 IR 装进内存，是内存杀手，开发期没有任何必要开。
- **codegen-units 调大**，不是调小。这个方向容易记反：`codegen-units = 1` 会把内存需求顶上去，调大到 256 反而省内存（代价是运行时性能，开发期无所谓）。
- **换链接器**。servo 在 Linux 上装了 lld 就会默认用它，确认一下 configure 输出里的 `checking for linker... lld`；Windows 上是 `export LINKER=lld-link`。链接是内存峰值的高发时刻，换 lld 或 mold 两头都省。
- **加上 swap/zram 兜底**。目的不是变快，是把"直接 OOM"变成"慢一点但能跑完"。

### 缩小重编的爆炸半径

改上游库最疼的地方在于：它坐在下游依赖链的根部，一改签名，下游全部重编。

- **优先"加"，而不是"改"**。加一个新类型、一个新 trait 的默认方法、一个新字段，只有相关 crate 重编；改一个被广泛使用的函数签名，全下游重编。同一个功能，两种改法代价能差一个数量级。
- **先把公共 API 定稳，再改实现**。cargo 的增量粒度是 crate：不动公共 API，就只重编当前这个 crate。
- **别 `cargo clean`，别乱动 workspace 的 `Cargo.toml` / `Cargo.lock`**。一改 fingerprint，全废。
- **sccache**（mach 里配 `[build] ccache = 'sccache'`）。但要清楚它的作用域：它救的是**清空 target、切分支、换构建配置**这类场景，而且命中率取决于依赖图稳不稳；它**救不了你的编辑回路**——改一个上游 crate 会让所有下游全部 cache miss。另外注意版本，老版本 sccache 在 cache miss 时会把增量编译拖慢好几倍（有人实测 1m53s 对 37s）。
- `target/` 放快盘上，排除出杀毒软件的实时扫描。

### rust-analyzer 那笔账

你说"16G 不够用，IDE 还有开销"——这两笔账其实可以分开算：

- **关掉 build script 重算**：`rust-analyzer.cargo.buildScripts.enable = false`。RA 为了跑 build.rs 会额外起一套 cargo，内存差不多翻倍，而 servo 这种项目你多半用不上这个精度。
- **给 RA 单独的 target 目录**：`rust-analyzer.cargo.targetDir`。
- **改掉它的 check 命令**：`rust-analyzer.check.overrideCommand` 指到 `./mach check` 或 `cargo check --message-format=json`。
- **最重要的一条：别让 RA 的检查和你的大构建同时跑。** 两个十 G 级别的内存峰值叠在一起必然爆。跑 `./mach build` 的时候把 RA 停掉（VSCode 里禁用扩展），这是零成本的。

### 硬件

说句直白的：**16G 对 servo 就是不够，这是配置问题，不是技巧问题。** 上面那些手段能把你从"跑不动"救到"跑得动"，但代价是墙上时间，而且 `-j` 调小和关优化会影响你验证真实行为的可信度。

32G 是现实底线，64G 才谈得上舒服。如果本地加不了内存，把构建放到远端大内存 Linux 机器上更划算——用延迟换内存。这个方向你本来就有经验（见 {% post_link SSHFS服务器目录挂载至本地 %}）。

以上都还在"想试"阶段，等真的跑出东西再回来补结论。


