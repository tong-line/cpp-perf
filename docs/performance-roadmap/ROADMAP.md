# 性能工程学习与求职路线图（ROADMAP）

> 最后更新：2026-11-09
> 状态标记：✅ 已完成 ｜ 🔶 进行中 ｜ ⬜ 未开始
> 规则：只有完成"验收标准"中的全部条目，才能把阶段标为 ✅。未学内容不得标记为已掌握。

## 1. 定位

- 华东师范大学软件学院研二，密码学方向 → 转向 **C++ 系统软件与性能工程**。
- 核心技术方向：**C++ 性能观测与分析工具研发（Performance Observability & Profiling）**。
- 目标岗位：
  - C++ 性能分析工具研发 / Profiler 开发
  - CPU/GPU/NPU 性能分析
  - Linux 系统性能工程
  - AI 芯片系统软件（重点关注寒武纪 CNPerf/CNPAPI 生态）
  - 自动驾驶、机器人系统观测工具研发

## 2. 主线与优先级

**主项目：Hotspot（KDAB/hotspot，Linux perf GUI）。**
目标不是"读过源码"，而是**独立完成一个有实际价值的功能改造**（能拿去社区 review 的那种）。

工具研究优先级（严格按此顺序，不并行铺开）：

1. Hotspot —— 源码精读 + 二次开发（主项目）
2. Linux perf —— 一切数据的源头，必须先于 Hotspot 精读掌握用户侧
3. Perfetto —— trace 数据模型与 Trace Processor
4. LTTng / ros2_tracing —— 内核/用户态追踪、ROS 2 实时性分析
5. 厂商工具 —— VTune、Nsight Systems/Compute、uProf、CNPerf/CNPAPI（体系对照，不深陷）

覆盖行业：芯片（PMU/硬件计数器）、自动驾驶（端到端延迟/异构调度）、机器人（ROS 2/LTTng/控制周期）、通用 Linux（CPU/内存/线程/栈/Trace）。

## 3. 阶段规划

### P0 环境与 perf 最小闭环 🔶
- [x] 确认 Ubuntu 24.04 实验环境为 **OrbStack ARM64 VM（无 PMU 透传）**；E1a 已验证软件事件可用、硬件事件不支持且 tracepoint 目录缺失。Ubuntu 24.04 仓库无 KF6，因此不用于 Hotspot 构建。
- [x] 新建并确认当前开发环境：OrbStack Ubuntu 26.04 ARM64 VM；Hotspot 在此成功编译。
- [x] 用户否决 ThinkBook 双系统方案；PMU 方案调整为短租 x86-64 独立服务器优先，云实例作为备选，但最终机器尚未确定。
- [ ] 在选定的 x86-64 环境完成 E1b，确认 cycles 等硬件事件/PMU 是否可用。
- [x] 克隆 Hotspot 源码（`--recurse-submodules`）并成功编译（2026-11-08，Ubuntu 26.04 VM，ARM64）。
- [x] 完成无头数据链验证：Ubuntu 26.04 上 `perf record -e cpu-clock` 生成 perf.data，Hotspot `--exportTo` 成功导出（7.8M → 56K）。E1a 已验证 Ubuntu 24.04 VM 的软件事件可用；`perf stat` / `perf report` 的完整实测仍需在后续实验中记录。
- [ ] GUI 打开同一份 perf.data（待显示问题解决：XQuartz 方案备用中）
- 验收（修订）：✅ 数据链路已验证（--exportTo 无头走通）；GUI 视图观察推迟到显示修复后，不阻塞 P1。

### P1 perf 与采样原理 ⬜
- [ ] `perf record` 参数体系：`-e`、`-g`、`--call-graph fp/dwarf`、`-F`、`-p`
- [ ] 核心硬件事件：cycles / instructions / cache-misses / branch-misses，理解 IPC
- [ ] `perf_event_paranoid` 权限体系；PMU 采样原理（计数器溢出中断、采样偏差/skid 概念）
- [ ] `perf script` 与原始数据观察
- 验收：EXPERIMENTS.md 中 E1–E4 完成，每条有可重复命令和真实数据。

### P2 CPU 微架构补课 🔶（贯穿 P1–P3）
- [ ] 流水线、乱序执行（OoO）、分支预测
- [ ] 缓存层次（L1/L2/L3、line、局部性）、TLB
- [ ] 与计数器一一对应：什么代码模式导致什么计数器升高
- 验收：给定"IPC 低 + cache-miss 高"的 profile，能列出完整的原因假设链并用实验验证。
- 教学方式：从软件层观察入手，一次一个知识点，配小实验。

### P3 Hotspot 架构精读 ⬜
- [ ] 第一步：仓库结构、CMake 组织、子模块内容
- [ ] 第二步：perfparser 进程（perf.data 解析 → 符号解析 → 事件流输出）
- [ ] 第三步：GUI 侧接收、核心数据结构（Data::*）
- [ ] 第四步：Bottom-Up / Top-Down / Caller-Callee / FlameGraph 模型构建
- [ ] 第五步：视图层与 Qt model/view、KDDockWidgets 布局
- [ ] 第六步：符号解析、sysroot/离线分析、debuginfod
- 验收：HOTSPOT_ARCHITECTURE.md 升级为"已验证版"；能独立画出完整数据流图（含线程关系）。

### P4 Hotspot 功能改造 ⬜
- 选题候选（届时按价值与难度评估）：
  - 将 hotspot 数据导出为 Perfetto 可打开的格式（README 明确说"planned, patches welcome"）
  - 灵活的 group-by / 列自定义（对应 perf report 缺失能力）
  - branch-stack / branch-history 支持评估（README 列为未支持）
- 流程：issue 沟通 → fork/分支 → 小步提交 → 社区 review。
- 验收：一个提交到上游的 PR（被 merge 或完整走完 review 闭环）+ 一篇复盘。

### P5 Perfetto ⬜
- [ ] trace 数据模型、Trace Processor（SQL 查询）、UI
- 验收：能把一份系统 trace 用 SQL 回答一个延迟问题。

### P6 LTTng / ros2_tracing ⬜
- [ ] 内核/用户态 tracepoint、CTF 格式、低开销采集
- [ ] ROS 2 回调/消息延迟分析
- 验收：在一个 ROS 2 示例上量化端到端消息延迟并定位瓶颈。

### P7 厂商工具体系对照 ⬜
- [ ] VTune（TMA / 微架构分析）、Nsight Systems vs Compute、uProf（IBS）
- [ ] CNPerf/CNPAPI：文档研究 + 概念映射到已掌握的 perf/Nsight 体系
- 验收：TOOLCHAIN.md 中形成"同一问题在四家工具里分别怎么做"的对照表。

### P8 求职冲刺 ⬜
- [ ] 简历项目叙事：perf→hotspot 改造→（perfetto/lttng）完整观测链能力
- [ ] 面试准备：采样原理、微架构、C++ 对象模型与并发、系统调试
- 时间锚点：**三年制，2028 年毕业**。两个求职窗口：
  - 主窗口：**2027 暑期实习**，优先投接受 2028 年毕业生的岗位；部分项目可能提前开放并滚动筛选，不等到 2027 年 2 月才开始关注。按岗位要求核对毕业时间，不能只看“2027 实习”等项目名称。
  - 兜底/正式窗口：**2027 秋招**（2027 年 9 月前后）；研三阶段也可先以日常实习切入（作品集 ready 即可，无严格窗口）
- 求职与学习并行：P0–P4 是能力建设路线，不是开始投递的前置门槛；发现合适岗位就先核对资格、投递并根据反馈补短板。
- **即时岗位线索（检索日期：2026-11-09；投递状态需以官方页面实时状态为准）**

  | 优先级 | 岗位 | 匹配与资格判断 | 下一步 |
  |---|---|---|---|
  | 1 | [Google 软件工程实习生（Summer 2027，上海）](https://www.google.com/about/careers/applications/jobs/results/109864326646375110-software-engineering-intern-summer-2027) | 通用软件工程岗位，Linux/并行与分布式等经验相关；官方页面显示申请窗口预计至 **2026-11-15**，滚动筛选。要求本科/硕士在读且处于倒数第二学年、2027 年暑期全职 10–12 周、英语沟通，并提交英文简历和成绩单。按目前“三年制、研二、预计 2028 毕业”的信息初步匹配，但须逐项核实申请表资格。 | **优先检查并尽早准备投递**；确认暑期可全职实习及英文材料。 |
  | 2 | [NVIDIA Software Engineering Intern - 2027（Developer Tools，上海）](https://jobs.nvidia.com/careers/job/893398124619) | 与性能分析、调优及 GPU Profiler 工具开发高度相关；官方岗位页未在本次检索摘要中确认毕业时间条件和截止日期。 | 打开官方页面核实届别、实习时长和申请状态，再决定投递。 |
  | 暂缓 | [字节跳动系统软件实习生-服务器（上海）](https://jobs.bytedance.com/campus/position/7595141804321556789/detail?sourceJobId=7600288827580483845) | 方向高度相关，但当前职位说明面向 **2027 届**；预计 2028 毕业者可能不符合该岗位届别。 | 除非官方页面或招聘方确认接受 2028 届，否则不优先投入。 |
- 简历素材（尚待个人简历整合，内容仅基于已记录的实验）：**Built KDAB Hotspot 1.6.80 from source on Ubuntu 26.04 ARM64 and validated a software-event profiling pipeline by recording a 200M-iteration workload with `perf` and exporting the 7.8 MB `perf.data` to a 56 KB perfparser file.**

## 4. 明确不做（防跑偏）

- ❌ mini-iceoryx / MiniProfiler / MiniAllocator（已放弃，不再重启）
- ❌ 脱离真实使用场景的自研轮子
- ❌ 在 Mac 上伪装 Linux/PMU 实验（见 EXPERIMENTS.md 环境矩阵）

## 5. 核心资料

| 资料 | 用途 |
|---|---|
| https://github.com/KDAB/hotspot （README / HACKING.md） | 主项目文档（2026-11-08 已抓取） |
| `man perf-record` / `man perf-stat` / `perf list` | 第一手 perf 文档 |
| MIT 6.172 | ✅ 已完成 |
| Brendan Gregg 站点（perf/火焰图） | perf 实战参考 |
| Perfetto docs | P5 资料 |

> 本文件每次阶段状态变化时更新；不追进度速度，追验收标准。
