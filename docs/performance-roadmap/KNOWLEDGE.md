# 知识地图（KNOWLEDGE）

> 最后更新：2026-11-08
> 规则：✅ = 能给别人讲清楚 + 有实验/代码证据；🔶 = 学了概念但没验证；⬜ = 未掌握。每次学习后更新。

## A. 已具备基础 ✅/🔶

| 领域 | 内容 | 深度 | 证据 |
|---|---|---|---|
| C++ | C++20 基础 + STL | ✅ 能写常规代码 | 课程/自学；模板元编程、并发尚未系统用过 🔶 |
| C 与构建 | C 基础、Makefile、编译和链接流程 | ✅ 入门完成 | 自学 |
| 汇编 | x86-64 入门：通用寄存器、RIP 相对寻址、movq、内存寻址模式 | ✅ 入门 | 自学 |
| 微架构概念 | CPU 寄存器、浮点数、SIMD 基础概念 | 🔶 概念级，无实验 | 自学 |
| 性能工程 | MIT 6.172 Performance Engineering | ✅ 课程完成 | 课程 |
| Linux | 基础操作 | ✅ 入门 | 日常使用 |

## B. 学习中 🔶

- CPU 微架构：正在从软件层逐步深入（下一步：流水线 → 缓存层次 → 分支预测）。

## C. 未掌握 ⬜（按对主线的重要性排序）

1. **Linux perf 实战**：record/report/stat/script、call-graph 模式（fp vs dwarf）、事件选择、`perf_event_paranoid`。→ 一切分析工具的数据源头。
2. **采样与 PMU 原理**：PMU 是什么、计数器溢出中断如何产生样本、采样频率与偏差（skid）、固定计数器 vs 可编程计数器。→ 面试高频 + hotspot 数据语义的前提。
3. **ELF / DWARF / 符号体系**：symtab/dynsym、DWARF 调试信息、内联函数展开、栈回溯三种途径（frame pointer / .eh_frame / DWARF CFI）、kallsyms、split debug 文件、debuginfod。→ hotspot 符号解析与栈回溯（libdw）的直接前提。
4. **调用栈模型**：调用栈如何在采样中被捕获、栈截断（`--call-graph dwarf,SIZE`）对 Top-Down/FlameGraph 的影响。
5. **Qt 基础**：信号槽、model/view 架构、QProcess、dock 组件。→ hotspot 是 Qt6 应用，视图层精读的前提。
6. **现代 CMake**：find_package、target_* 用法、git submodule 组织。→ 编译 hotspot 的前提（已有 Makefile 基础可迁移）。
7. **C++ 并发**：线程、原子操作、内存序。→ perfparser 多线程解析 + KDE ThreadWeaver 后台任务。
8. **内核观测基础**：syscall、tracepoint、perf_event 子系统的位置。→ P5/P6 的地基。

## D. 依赖关系图

```
                 CPU 微架构
                     │
                     ▼
              PMU / 硬件计数器 ──► 采样原理 (skid/中断)
                     │
                     ▼
              Linux perf (用户侧) ──► perf.data 格式
                     │                      │
                     ▼                      ▼
        ELF/DWARF/符号 ──► 栈回溯 ──► perfparser (解析+符号解析)
                     │                      │
                     ▼                      ▼
              Call Tree/FlameGraph ◄── Data 数据结构
                                            │
                          Qt model/view ◄───┤
                                            ▼
                                    Hotspot 视图层
```

学习顺序上：C 列表的 1→2→3 是硬依赖链，4 随 3 顺带，5/6/7 在进入 P3 视图层精读前补。

## E. 与 Hotspot 研读的知识映射

| Hotspot 模块（名称待源码验证） | 需要的知识 | 当前状态 |
|---|---|---|
| perfparser 进程：perf.data 解析 | perf.data 二进制格式、perf 文件头/事件 | ⬜ |
| perfparser 进程：栈回溯 | DWARF CFI、libdw（elfutils） | ⬜ |
| perfparser 进程：符号解析 | ELF symtab、kallsyms、debuginfod | ⬜ |
| GUI 进程：数据接收与存储 | 进程间管道协议（具体形式待验证）、数据结构设计 | ⬜ |
| GUI 进程：Bottom-Up/Top-Down/FlameGraph 模型 | 树聚合算法、增量构建 | ⬜ |
| GUI 进程：视图与交互 | Qt model/view、KDDockWidgets | ⬜ |
| 构建系统 | CMake、KF6、子模块 | ⬜ |

## F. 更新规则

- 每完成一个知识点（讲解 + 练习 + 实验证据），把对应条目从 ⬜/🔶 移到 ✅，并在 DEVELOPMENT_LOG.md 记录。
- 允许"🔶 概念级"存在，但不允许把概念级标成 ✅。
