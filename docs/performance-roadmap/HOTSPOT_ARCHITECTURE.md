# Hotspot 架构预研（HOTSPOT_ARCHITECTURE）

> 最后更新：2026-11-09
> ⚠️ **验证状态：源码已于 2026-11-08 克隆（含子模块），并在 OrbStack Ubuntu 26.04 ARM64 VM 成功编译；但本文架构梳理多数仍基于公开资料、构建记录和无头链路验证，尚未逐文件核验。**
> 标记：✅ 公开资料直接确认 ｜ 🔶 合理推断（待源码验证）｜ ⬜ 待研究
> 下一步从 §7 的 R1 开始，按阅读计划逐条验证并改写本文件。

## 1. 项目定位 ✅

- **"The Linux perf GUI for performance analysis"**——KDAB 的 Linux perf 图形化分析工具，目标是做出"perf 版的 KCachegrind"。
- 不负责采集（采集交给 perf），核心价值在**解析 + 符号化 + 可视化 + 离线分析**。
- License：GPL v2+。项目自身编译标准：C++17（不是 C++20——向它贡献代码时要遵循项目风格）。

## 2. 双进程架构 ✅（进程划分）／⬜（细节）

```
┌─────────────────────────┐         管道/IPC（具体形式待验证）        ┌──────────────────────────┐
│  hotspot (GUI 进程)      │ ◄──────────────────────────────────── │ hotspot-perfparser 进程   │
│  Qt6 应用                │      流式二进制事件（协议待验证）        │  - 解析 perf.data         │
│  - 数据接收与存储         │                                       │  - ELF 符号解析 (libelf)  │
│  - 聚合模型构建           │                                       │  - 栈回溯 (libdw/DWARF)   │
│  - 各视图渲染             │                                       │  - demangle               │
└─────────────────────────┘                                       └──────────────────────────┘
        ▲                                                                      ▲
        │ 打开文件                                                              │ 运行/读取
        └────────────── perf.data ←── perf record（采集由外部 perf 完成）────────┘
```

- GUI 可执行文件叫 `hotspot`；解析器是独立的 libexec 二进制 `hotspot-perfparser` ✅（HACKING.md 的 debuginfo 路径 `usr/libexec/hotspot-perfparser` 直接证实）。
- perfparser 源自 **Qt Company 为 Qt Creator 开发的 perfparser 工具**，hotspot 复用了它 ✅（README "Qt Creator" 一节原文）。
- 🔶 推断：两进程间用流式二进制协议（`--exportTo` 能导出同格式 `.perfparser` 文件，说明该事件流可落盘回放）。协议细节待读源码。
- 🔶 推断：解析放独立进程的好处——解析崩溃不连累 GUI、解析可流水化、C++ 解析代码与 Qt Creator 上游同源可跟版。

## 3. 核心依赖 ✅（来自 HACKING.md）

| 依赖 | 作用（推断处标 🔶） |
|---|---|
| CMake ≥ 3.16、C++17 编译器 | 构建 |
| Qt ≥ 6.0（含 qt6-svg） | GUI 基础 |
| KDE Frameworks 6：threadweaver, i18n, configwidgets, coreaddons, itemviews, itemmodels, kio, solid, windowsystem, notifications, iconthemes, parts, archive(可选) | 🔶 ThreadWeaver 大概率用于后台解析/聚合任务 |
| KDDockWidgets | 🔶 可停靠窗口布局（README 截图含 dockwidget layouts） |
| kgraphviewer | 🔶 调用图（call graph）渲染 |
| qcustomplot | 🔶 时间线/图表绘制 |
| libelf / libdw (elfutils) / libdwarf / libdebuginfod | ELF 解析、DWARF 栈回溯、调试符号在线下载 ✅（README 明确 unwind 依赖 libdw） |
| libzstd | 🔶 perf.data 的 zstd 压缩支持 |
| gettext, extra-cmake-modules | i18n、KDE 构建辅助 |

- 克隆命令：`git clone --recurse-submodules https://github.com/KDAB/hotspot.git`——子模块已验证 ✅（2026-11-08 编译时确认）：`3rdparty/perfparser`（KDAB/perfparser 独立仓库）+ `3rdparty/PrefixTickLabels`。**"perfparser 是外部独立项目、以子模块引入"的推断实锤。**
- 仓库自带 autotests（构建时产出 `tst_callgraphgenerator` 等测试目标）🔶——问题 9 的部分线索。
- 环境验证：Ubuntu 24.04（noble）仓库未提供所需 KF6 包，因此该环境未用于构建；Ubuntu 26.04 仓库依赖安装成功，Hotspot 1.6.80 在 ARM64 上编译到 100%（2026-11-08）。

## 4. 功能视图清单 ✅（README 截图与功能章节）

| 视图/功能 | 说明 |
|---|---|
| Summary 页 | 总览：样本数、进程/线程/事件统计 |
| Bottom-Up | 自底向上调用聚合 |
| Top-Down | 自顶向下调用树（依赖完整栈，栈截断会破坏它） |
| Caller / Callee | 调用者/被调用者关系 |
| FlameGraph | 火焰图（内联函数有更深的边框标示） |
| Time Line | 时间线：按时间/进程/线程过滤，联动上方所有数据视图，可缩放到单样本 |
| 反汇编视图 | **每条指令的成本标注**，汇编-源码行颜色对应，双击跟进函数 |
| Record 页 | 从 GUI 启动 perf record（启动新程序或 attach 已有进程），含 off-CPU 复选框 |
| Off-CPU 分析 | 基于 `sched:sched_switch` tracepoint + `--switch-events`，给出线程不在 CPU 上的时间及原因（IO/缺页/sleep/futex/抢占） |
| 离线/嵌入式分析 | `--sysroot`、`--kallsyms`、`--appPath`、`--extraLibPaths`、`--debugPaths`、`--sourcePaths`、`--arch` 一整套符号定位机制 |
| 导入/导出 | File > Save As 导出自含的 `.perfparser` 文件（格式未稳定，仅同版本可读）；CLI `--exportTo` 可无 GUI 转换 |
| debuginfod | 支持在线下载缺失调试符号（设置或 `DEBUGINFOD_URLS`） |

已知不支持（README 明示）：`--itrace`、`--mem-mode`、`--branch-stack`、`--branch-history`；表格列硬编码（group-by 只有基础实现）。
→ 这些"官方不做的"正是 P4 功能改造的候选池。

## 5. 数据流主线（待源码验证的骨架）🔶

```
perf record (--call-graph dwarf) ─► perf.data
   │
   ▼
perfparser: 读 perf.data → 解析文件头/事件/样本 → 按映射表(mmap 事件)归属二进制
   │            → 符号解析(ELF/kallsyms/demangle) → 栈回溯(libdw) → 归一化调用栈
   ▼
事件流(协议待验证) ─► GUI 侧 PerfParser 接收器 → Data 核心结构(命名待验证)
   │
   ├─► Summary 模型            ├─► Bottom-Up 聚合
   ├─► Top-Down 聚合           ├─► Caller/Callee 聚合
   ├─► Time Line 事件序列      └─► FlameGraph（多由聚合树派生）
   ▼
Qt 视图渲染（可停靠布局；反汇编视图按需读 ELF/objdump）
```

## 6. 源码阅读前的问题清单（P3 的导览问题）

1. perfparser 与 GUI 之间的协议长什么样？消息类型有哪些？⬜
2. GUI 侧核心数据结构是什么（样本、符号、调用栈如何表示）？⬜
3. Bottom-Up / Top-Down 树是解析完成后一次性构建，还是流式增量构建？⬜
4. 大文件（GB 级 perf.data）的内存策略：全量驻留还是按需加载？⬜
5. 符号解析的缓存与查找顺序（appPath → extraLibPaths → sysroot → debuginfod）如何实现？⬜
6. ThreadWeaver 具体用在哪些任务？哪些步骤是多线程的？⬜
7. qcustomplot / kgraphviewer 分别被哪个视图使用？⬜
8. 子模块里到底有什么？⬜
9. 测试如何组织（有无 autotests/、如何构造 perf.data 测试样本）？⬜
10. 时间过滤是如何作用于各模型的（过滤在模型层还是数据层）？⬜

## 7. P3 阅读计划（从 R1 开始执行）

| 步骤 | 内容 | 产出 |
|---|---|---|
| R1 | 仓库树 + 顶层 CMakeLists + 子模块清单 | 目录结构注释版 |
| R2 | perfparser 进程：入口、解析循环、输出协议 | §5 数据流左半边验证 |
| R3 | GUI 接收端 + Data 核心结构 | 核心类卡片（每个类：职责/关键字段/协作类） |
| R4 | 聚合模型（Bottom-Up 构建算法） | 问题 3/4 的答案 |
| R5 | 视图层（FlameGraph 绘制、Time Line 联动） | 问题 10 的答案 |
| R6 | 符号解析与 sysroot 体系 | 问题 5 的答案 |
| R7 | Record 页与 perf 进程管理 | 记录页逆向 |

阅读纪律：先架构后细节；每步先预测实现再看代码；发现与本文档不符时**立即改写本文档**并在 DEVELOPMENT_LOG.md 记录。
