# 开发日志（DEVELOPMENT_LOG）

> 记录格式：日期 ｜ 类型（环境/阅读/实验/开发）｜ 做了什么 ｜ 产出 ｜ 下一步
> 每次源码阅读、功能开发、调试之后都要追加一条；宁可短，不可断。

---

## 2026-11-09 ｜ 实习求职启动

**做了什么**
- 按预计 2028 年毕业筛选公开岗位，发现 Google 上海 Summer 2027 软件工程实习（页面显示申请窗口至 2026-11-15，滚动筛选）及 NVIDIA 上海 Developer Tools 实习（GPU 性能分析/Profiler 工具方向）。
- 核对届别：字节跳动当前服务器系统软件实习岗位注明面向 2027 届，不能仅凭岗位方向匹配就假定 2028 届可投。
- 将岗位线索、申请检查点和 Hotspot/perf 实验经历的简历素材写入 ROADMAP.md。

**下一步**
- 优先核对 Google 岗位资格并准备英文简历/成绩单；核对 NVIDIA 岗位的毕业时间要求与截止日期。
- 继续 perf 入门实验；VM 输出尚待实际运行后再分析，不预设数据结论。

---

## 2026-11-09 ｜ 文档状态同步

**做了什么**
- 按既有实验记录统一环境状态：Ubuntu 24.04 OrbStack VM 已完成 E1a 后弃用；Ubuntu 26.04 OrbStack VM 是当前 Hotspot 编译环境。
- 修正 PMU 方案：ThinkBook 双系统方案已否决；短租 x86-64 独立服务器优先，但尚未租用或完成 E1b 验证。
- 同步 E4 进度为无头数据链已完成、GUI 视图验证待完成，并修正 Hotspot 架构文档的源码状态。

**产出**
- 更新 EXPERIMENTS.md、ROADMAP.md、HOTSPOT_ARCHITECTURE.md，避免环境与进度状态冲突。

**下一步**
- 继续排查 Ubuntu 26.04 VM 的 GUI 显示；之后补做 E4b 视图观察，并在选定的 x86-64 环境完成 E1b。

---

## 2026-11-08 ｜ 环境与项目初始化

**做了什么**
- 确认工作区 `/Users/tongzhaolin/c++perf` 为空目录：**hotspot 源码尚未克隆**（遵守规则未擅自下载）。
- Mac 本机工具链盘点：macOS 27.2 (arm64)、Apple Clang 21.0.0、CMake 4.4.4、GNU Make 3.81、Homebrew + Qt 6.11.2 (brew)、git 2.54、Docker 29.8.0；无 ninja。
- 抓取并研读 hotspot 公开资料：
  - README：功能全景（Summary/Bottom-Up/Top-Down/Caller-Callee/FlameGraph/TimeLine/反汇编/off-CPU/离线分析/导入导出）；perfparser 来自 Qt Company（Qt Creator）；栈回溯依赖 elfutils 的 libdw；明确不支持的 perf 能力（itrace/mem-mode/branch-stack/branch-history）；GPL v2+。
  - HACKING.md：构建依赖（Qt6、KF6 全家桶、KDDockWidgets、kgraphviewer、qcustomplot、libelf/libdw/libdebuginfod、libzstd）、`--recurse-submodules` 构建、C++17 标准、AppImage 调试方法。
- 创建 `docs/performance-roadmap/` 六份文档并互相引用。

**产出**
- 六份初始文档；hotspot 架构预研（含 10 个待验证问题清单）；实验规划 E1–E5。

**结论 / 下一步**
- Mac 只承担"读码、写码、Git"；perf/PMU/hotspot 编译全部在 Ubuntu 24.04 进行。
- 阻塞项：① Ubuntu 24.04 是真机还是虚拟机待确认；② hotspot 源码待克隆编译。
- 下一条日志应该是：环境确认 + E1 探测，或克隆编译记录。

---

## 2026-11-08（第 2 条）｜ 环境确认与计划修订

**做了什么**
- 确认 Ubuntu 24.04 = **OrbStack 虚拟机**（Mac 内 ARM64 VM）。关键影响：Apple 虚拟化**无 PMU 透传** → perf 只能做软件事件工作流练习，PMU 数值实验必须另找真机。
- 修订 EXPERIMENTS.md 环境矩阵：OrbStack 定位为"hotspot 编译开发 + 源码阅读主战场"；新增 ThinkBook 双系统 Ubuntu 作为 PMU 真机候选方案（最低成本）；云 x86-64 降级为构建验证机（云主机本身虚拟化，PMU 大概率不下发，须实测）。
- E1 拆分：E1a（OrbStack，立即可做）/ E1b（真机，硬件事件）。
- 评估学习时间投入：每天 8 小时，更新求职时间线（见 ROADMAP P8）。

**结论 / 下一步**
- 立即可做：OrbStack 里执行 E1a + 克隆编译 hotspot（任务 1 解除阻塞）。
- 待决策：ThinkBook 是否双系统装 Ubuntu（PMU 真机方案）。
- 下一条日志：E1a 探测结果 + hotspot 编译记录。

---

## 2026-11-08（第 3 条）｜ E1a 部分完成，PMU 方案调整

**做了什么**
- E1a（OrbStack）：内核 7.2.12-orbstack、aarch64、10 vCPU、paranoid=2；`perf list` 无输出，perf 工具待安装确认。结果记入 EXPERIMENTS.md。
- 用户决策：**ThinkBook 双系统否决**。PMU 真机方案改为候选：短租 x86-64 独立服务器（1–2 个月）或实测可下发 PMU 的云实例；第 3 周前定。

**下一步**
- 安装 perf 并补测软件事件 → 安装 hotspot 依赖并编译（P0 主线）→ hotspot GUI 显示验证。

---

## 2026-11-08（第 4 条）｜ 依赖排障：定位到 24.04 无 KF6，决定新建 25.10/26.04 VM

**做了什么（调试链：报错 → 假设 → 验证）**
- 现象：apt 装 KF6 全部 `Unable to locate package`，CMake 连锁报 Qt6 找不到。
- 假设 1：universe 未启用 → 验证：启用后仍 MISSING → 推翻。
- 假设 2（确认）：**Ubuntu 24.04 (noble) 仓库根本没有 KF6**（KF6 自 24.10 进入 Ubuntu；noble 是 KF5 时代），而 hotspot master 要求 Qt6+KF6。HACKING.md 的 apt 命令面向新发行版。
- 另一问题：子模块克隆 GnuTLS 断连（国内 GitHub 链路），已设 `http.version HTTP/1.1`，待重试。
- 决策：OrbStack 新建 Ubuntu 26.04（fallback 25.10）虚拟机作为 hotspot 开发环境；24.04 VM 弃用（perf 软件事件结论仍有效）。
- 顺带确认：hotspot 子模块 = `3rdparty/perfparser`（KDAB/perfparser，架构文档推断被验证）+ `3rdparty/PrefixTickLabels`。

**下一步**
- 新 VM：依赖安装 → 克隆（含子模块）→ 编译 → GUI 显示验证 → P0 验收。

---

## 2026-11-08（第 5 条）｜ hotspot 编译成功（P0 关键里程碑）

**做了什么**
- OrbStack 新 VM（ubuntu-2604，Ubuntu 26.04，perf 7.0.14）上完成：
  - 系统初始化（sysctl.d 持久化 paranoid=-1、perf 自动软链、http.version HTTP/1.1）；
  - KF6 全套依赖一次装齐（26.04 仓库含 libkddockwidgets-qt6-dev / kgraphviewer-dev，无需源码编译）；
  - `git clone --recurse-submodules` 成功（子模块 perfparser + PrefixTickLabels 完整）；
  - **cmake + make 编译到 100%，产出 `~/build-hotspot/bin/hotspot`**（ARM64 构建）。
- 附带验证：构建系统自动编出了 `tst_callgraphgenerator` 等测试目标 → 仓库自带 autotests（HOTSPOT_ARCHITECTURE 问题 9 的线索）。

**下一步**
- GUI 显示验证（OrbStack X11 转发）→ E4 闭环（录 perf.data 并用自编译 hotspot 打开）→ P0 验收。

---

## 2026-11-08（第 6 条）｜ P0 实质验收：E4 无头闭环跑通

**做了什么**
- E4 无头部分：bench.c（200M 循环）→ `perf record -e cpu-clock --call-graph dwarf` → perf.data 7.8M → 自编译 hotspot `--exportTo` → out.perfparser 56K，全链无报错。
- 遗留：GUI 显示（自定义 orb create 机器无 DISPLAY 注入，XQuartz 方案备用）；`perf stat` 尚未在这台 26.04 上复跑（下一课内容）。

**下一步**
- P1 第一课（Counting vs Sampling）以本份 perf.data 为素材开讲；GUI 支线继续修。

<!-- 追加新日志时写在上方分隔线之前，保持最新在上 -->
