# 实验记录（EXPERIMENTS）

> 最后更新：2026-11-09
> 规则：每条实验必须完整走模板；没有可复现命令 + 真实数据的结论一律不算数。

## 0. 环境能力矩阵（先想清楚在哪做实验）

| 环境 | 架构 | perf | 真实 PMU | 编译 hotspot | 定位 |
|---|---|---|---|---|---|
| MacBook Air M5（本机，macOS 27.2） | arm64 | ❌ | ❌ | ❌（Linux-only 项目） | 代码阅读/编辑、Git、C++ 语法练习、ARM64 Docker 容器（无 PMU） |
| ThinkBook（Windows） | x86-64 | ❌（宿主） | ❌ | ❌ | 备用机 |
| OrbStack Ubuntu 24.04 VM（Mac 内，E1a 已完成；后因缺 KF6 停用） | arm64 | ✅ 软件事件 | ❌（Apple 虚拟化无 PMU 透传） | ❌（24.04 仓库无 KF6，未用于构建） | E1a 环境/权限探测；perf 软件事件工作流 |
| OrbStack Ubuntu 26.04 VM（Mac 内，当前开发环境） | arm64 | ✅ 软件事件（perf 7.0.14；E4 使用 `cpu-clock`） | ❌（虚拟机，无可采信的 PMU 数值） | ✅（Hotspot 1.6.80） | **Hotspot 编译开发 + 软件事件 perf 工作流** |
| 国内云 x86-64 服务器（候选） | x86-64 | ✅（待具体实例验证） | ⚠️ 可能不下发 PMU，须实测 | ✅（构建验证候选） | x86-64 构建/perf 工作流；E1 验证前不采信 PMU 数值 |
| 短租 x86-64 独立服务器（候选，尚未租用） | x86-64 | 待实测 | 待 E1b 验证 | ✅（计划用途） | **当前优先考虑的 PMU 真机方案** |
| Docker（Mac 内，Linux VM 之上） | arm64（可模拟 x86-64，慢） | 部分 | ❌（无透传） | 可编译，性能实验无意义 | 仅构建验证 |

**铁律：**
1. OrbStack VM / Mac / Docker 一律不能替代真实 x86-64 PMU 实验（Apple 虚拟化不透传 PMU，计数器不可用或不可信）。
2. 云服务器本身也是虚拟机：先跑 E1 探明硬件事件是否下发，探不明就只当 x86-64 构建验证机。
3. **凡是涉及 PMU 数值（cycles/cache-miss/branch-miss/IPC）的结论，只能出自经验证可用 PMU 的真实硬件**。ThinkBook 双系统方案已否决；当前优先考虑短租 x86-64 独立服务器，最终环境待定。
4. OrbStack VM 是 ARM64：hotspot 开发与软件事件 perf 完全没问题，但涉及 x86-64 汇编级分析（反汇编视图对照你的 x86-64 汇编学习）时要用真机。

## 1. 实验模板

```
### E<N> <标题>　[状态: ⬜规划/🔶执行中/✅完成]
- 日期：
- 目的：
- 环境：（哪台机器、CPU 型号、内核版本）
- 前置条件：（权限、依赖）
- 步骤与命令：
- 关键数据：（原始输出摘录）
- 分析：（数据说明了什么，与预期是否一致）
- 结论：（一句话，写回对应文档）
- 复现脚本：（如有）
```

## 2. 已规划实验

### E1 环境与权限探测 🔶（E1a 完成；E1b 待做）
- 目的：摸清 OrbStack 环境及候选 x86-64 主机上 perf 的可用边界。
- E1a（已完成）：OrbStack Ubuntu 24.04 的软件事件可用性、PMU 和 tracepoint 限制。
- E1b（待做）：在选定的 x86-64 云实例或短租独立服务器上验证硬件事件与 PMU。
- 命令草案：`uname -a`、`lscpu`、`cat /proc/sys/kernel/perf_event_paranoid`、`perf list | head`、`perf stat true`。
- 判定：paranoid 值决定能做什么级别的采样；虚拟化下 `perf list` 可能缺硬件事件。

### E2 IPC 对照基准（计数器入门）⬜
- 目的：用 `perf stat` 直观感受"访存局部性对 IPC 的影响"，把微架构概念和计数器挂钩。
- 设计草案：同一程序两种访存模式（顺序遍历 vs 大步长跨行遍历），对比 cycles / instructions / cache-misses / IPC。
- 需要先完成 E1。数据出来后写回 KNOWLEDGE.md 微架构条目。

### E3 三种调用栈回溯对比 ⬜
- 目的：理解 `--call-graph fp / dwarf` 的取舍（栈完整性 vs 开销 vs 文件大小），这是读懂 hotspot 栈回溯代码的体验基础。
- 设计草案：带内联和深递归的测试程序，`-fno-omit-frame-pointer` 与默认 `-O2` 各录一次，`perf report` 对比栈深与符号名。

### E4 perf → hotspot 闭环 🔶（无头部分完成；GUI 部分待做）
- 目的：以**用户**身份完整走一遍 hotspot 工作流（这是精读源码前必须有的用户视角）。
- E4a（已完成）：在 Ubuntu 26.04 VM 录制软件事件 perf.data，并由自编译 hotspot 运行 `--exportTo`。
- E4b（待做）：解决 GUI 显示问题后打开同一份数据，观察 Summary/Bottom-Up/Top-Down/FlameGraph/TimeLine/反汇编视图及交互差异。

### E5 off-CPU 分析初体验 ⬜（E4 之后）
- 目的：体验 README 的 off-CPU 命令（`-e cycles -e sched:sched_switch --switch-events --sample-cpu --call-graph dwarf`），理解 on-CPU 与 off-CPU 两种视角。

## 3. 已完成实验

### E4a perf → hotspot 闭环（无头部分）　[✅ 完成]
- 日期：2026-11-08
- 环境：OrbStack ubuntu-2604 VM，perf 7.0.14，自编译 hotspot 1.6.80（ARM64）。
- 命令与数据：
  - 测试程序 `bench.c`（200M 次循环 + volatile sink），`gcc -O2 -g`；
  - `perf record -e cpu-clock --call-graph dwarf ./bench` → **perf.data 7.8M**；
  - `hotspot --exportTo /tmp/out.perfparser perf.data` → **out.perfparser 56K**，无报错。
- 结论：
  1. **数据链路全通**：perf.data → perfparser（解析+符号+DWARF 回溯）→ GUI 进程接收 → 导出。P0 实质验收完成；
  2. 140:1 的体积比 = DWARF 模式复制的原始栈内存 vs 符号化后的归一化调用栈，直观印证"perf.data 不自含 / perfparser 格式自含"；
  3. GUI 打开同一文件待显示问题解决后补（不影响链路结论）。
- 待续：GUI 视图观察（E4b）；真机上用硬件事件 cycles 重录对比（E1b 后）。

### E1a OrbStack 环境探测　[✅ 完成]
- 日期：2026-11-08
- 目的：摸清 OrbStack Ubuntu 上 perf 的可用边界。
- 环境：OrbStack Ubuntu 24.04，内核 `7.2.12-orbstack-00394-gbecdd6130068`，aarch64，10 vCPU。
- 关键数据：
  - `perf_event_paranoid` 初始 2，已放开为 -1
  - perf 工具：Ubuntu 的转发脚本按内核版本找人，OrbStack 定制内核无匹配 → 修复：软链 `ln -sf /usr/lib/linux-tools/6.8.0-149-generic/perf /usr/local/bin/perf`，`perf version 6.8.12` 可用
  - **软件事件全部可用**：task-clock、context-switches、cpu-migrations、page-faults 等
  - **硬件事件全部 `<not supported>`**：cycles / instructions / branches —— VM 无 PMU 的直接证据（验证了环境矩阵的预判）
  - `Error: failed to open tracing events directory` —— tracepoint 目录缺失（内核配置/未挂载），**off-CPU（sched:sched_switch）类实验在本 VM 不可行**，真机再补
- 结论：
  1. OrbStack 定位确认 = hotspot 开发 + perf **软件事件**工作流练习；
  2. 一切 PMU 数值与 tracepoint 实验延后到经验证的硬件环境；短租 x86-64 独服优先考虑，最终方案待定；
  3. GUI 显示（X11/Wayland/XQuartz）仍待解决；Hotspot 已在 Ubuntu 26.04 VM 编译成功。

## 4. 数据归档约定

- perf.data 等大二进制**不入 git**，只记录命令、关键输出摘录与脚本。
- 可复现脚本放 `experiments/` 目录（待创建），文档只引用。
