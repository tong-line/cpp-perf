# 性能工具链对比（TOOLCHAIN）

> 最后更新：2026-11-08
> 学习状态：🔶 概念级 ｜ ⬜ 未学 ｜ ✅ 已掌握（须有实验证据）

## 1. 观测体系分层（先建立这张地图）

```
┌─ 分析层 ────────────────────────────────────────────────┐
│ 汇总/火焰图: perf report, Hotspot                        │
│ 时间线:      Perfetto UI, Nsight Systems, Hotspot timeline│
│ 深度计数:    VTune(TMA), Nsight Compute, CNPerf          │
└──────────────────────────────────────────────────────────┘
┌─ 采集层 ────────────────────────────────────────────────┐
│ 硬件计数器(PMU): perf_event → perf / VTune / uProf      │
│ 内核事件:        tracepoint, probe → perf / LTTng / eBPF │
│ 用户态:          sdt, instrumented SDK → CUPTI / CNPAPI  │
└──────────────────────────────────────────────────────────┘
```

两种基本范式（记住这对概念，贯穿全部工具）：
- **Counting（计数）**：确定性总量，如 `perf stat` 的 cycles/instructions。
- **Sampling（采样）**：按事件周期性打断并记录现场，统计意义上近似真实分布，如 `perf record`。

## 2. 主工具对比

| 工具 | 层级 | 采集机制 | 数据格式 | 可视化 | 典型场景 | 状态 |
|---|---|---|---|---|---|---|
| **perf** | 内核 perf_event 子系统 | 计数 + PMU 采样 + tracepoint | perf.data（二进制，**不自含**，需符号/ELF 配合） | perf report/script TUI | CPU 热点、on-CPU/off-CPU、事件统计 | 🔶 概念级 |
| **Hotspot** | perf.data 的 GUI 前端 | 不采集，解析 perf.data（内含 perfparser 进程） | perf.data / 自含的 *.perfparser 导出格式 | Summary、Bottom-Up/Top-Down、Caller-Callee、FlameGraph、Time Line、反汇编 | 把 perf 结果变成可交互分析 | ⬜ 主项目 |
| **Perfetto** | 全系统 trace | 多源 trace 汇聚（ftrace、系统 trace、SDK） | protobuf trace（**自含时间线语义**） | Web UI + trace processor（SQL） | 跨进程/跨设备时间线、端到端延迟 | ⬜ |
| **LTTng** | 内核 + 用户态 tracepoint | 低开销静态探针，CTF 格式 | CTF（需 Babeltrace 转换） | 第三方（常配 Trace Compass / Perfetto） | 实时系统、ROS 2、生产环境低开销追踪 | ⬜ |

补充（主线外，了解即可）：eBPF（bpftrace/BCC）是内核态可编程观测，与 perf 互补，面试常问；不在当前优先级内。

## 3. 厂商工具对照

| 厂商工具 | 对标关系 | 关键概念 | 状态 |
|---|---|---|---|
| Intel VTune | perf 的商业超集（x86 深度优化） | TMA（Top-down Microarchitecture Analysis）分层归因 | ⬜ |
| NVIDIA Nsight Systems | 全系统时间线（Perfetto 哲学相近） | CPU/GPU 交织时间线、NVTX 标注、CUPTI 采集 | ⬜ |
| NVIDIA Nsight Compute | 单 kernel 深度计数（对应 VTune 的 kernel 视角） | kernel 重放、硬件计数器分组 | ⬜ |
| AMD uProf | 对标 VTune | IBS（Instruction-Based Sampling，精确采样） | ⬜ |
| 寒武纪 CNPerf / CNPAPI | 体系上对标 Nsight Systems + CUPTI（**此映射是推测框架，以官方文档为准**） | 主机-设备时间线、设备计数器、API 层插桩 | ⬜ 文档研究未开始 |

> 研究厂商工具的目的不是记住按钮，而是理解**同一套观测范式在不同硬件上的表达**：计数器从哪来 → 怎么采样 → 数据模型长什么样 → 怎么归因。

## 4. perf 命令速查（随实验逐步补充真实输出）

```bash
# 计数（E2）
perf stat -e cycles,instructions,cache-misses,branch-misses ./app
# 采样（E3）
perf record --call-graph dwarf ./app      # DWARF 回溯，栈最全，文件大
perf record --call-graph fp ./app         # 帧指针回溯，需要 -fno-omit-frame-pointer
# 报告 / 原始数据
perf report
perf script | head
# 权限
cat /proc/sys/kernel/perf_event_paranoid   # 2: 禁内核剖析; 1: 禁CPU事件; 0: 禁raw tracepoint; -1: 全开
```

## 5. 学习顺序（与 ROADMAP 一致）

perf（数据源头）→ Hotspot（分析与 GUI）→ Perfetto（trace 数据模型）→ LTTng（trace 采集）→ 厂商工具（体系对照）。

理由：**先理解数据是怎么来的，再理解数据是怎么被加工和展示的**——这决定了你能不能做"工具研发"而不只是"工具使用"。
