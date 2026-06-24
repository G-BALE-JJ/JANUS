# JANUS 项目：研究计划与背景文档
> **供 AI Agent 参考使用** | 最后更新：2026-06-24
> 本文档汇总了 JANUS 项目的完整研究计划、架构设计、相关工作与发表策略。

---

## 1. 项目概述

**项目代号**：JANUS
**完整描述**：面向 LLM 推理的异构 RISC-V Golem 阵列 Prefill-Decode 分离 CIM 多核加速器

### 一句话摘要
> 我们将 LLM 推理的 Prefill 与 Decode 两个阶段分别部署在异构片上 CIM（存算一体）阵列（P-Golem 与 D-Golem）上，每个 Golem 配备独立的 RISC-V 核进行自治调度，从而实现去中心化调度与硬件级 KV Cache 管理。

### 五大核心创新点

1. **Golem 异构化**：将统一的 Golem CIM 阵列拆分为 P-Golem（GEMM 优化）和 D-Golem（GEMV 优化）
2. **去中心化 RISC-V 调度**：每个 Golem 内嵌基于 Vanadis 的 RISC-V 核，实现自治任务分发，无需中央控制器
3. **硬件级 PagedAttention**：KV Cache 页的分配与释放由 D-Golem RV 核直接在硬件层管理，消除软件开销
4. **自定义 ISA 扩展**：Prefill 使用 `golem.p.*` 指令族，Decode 与 KV 管理使用 `golem.d.*` / `golem.kv.*` 指令族
5. **专用 KV 迁移总线**：片上专用总线连接 P-Golem 与 D-Golem，KV 传输延迟达纳秒级（相比 GPU 方案的微秒~毫秒级）

---

## 2. 背景与动机

### 2.1 LLM 推理的两阶段特性

| 阶段 | 计算模式 | 瓶颈类型 | 底层操作 |
|------|---------|---------|---------|
| **Prefill（预填充）** | 处理输入 Prompt | 计算密集型（Compute-bound） | GEMM（矩阵×矩阵） |
| **Decode（解码）** | 自回归逐 Token 生成 | 内存密集型（Memory-bound） | GEMV（矩阵×向量） |

**核心问题**：两阶段混合运行在同一硬件上会造成严重干扰：
- Decode 的小 Batch 请求抢占 Prefill 的计算资源
- TTFT（首 Token 延迟）与 TPOT（每 Token 生成时间）同时恶化
- Decode 阶段 GPU 利用率通常低于 30%

### 2.2 现有方案的局限性

- GPU 显存带宽有限，KV Cache 碎片化严重
- 中心化调度器在动态请求负载下成为瓶颈
- 异构节点间（PCIe/NVLink）的 KV Cache 迁移延迟高（微秒~毫秒级）
- 软件层 PagedAttention 存在不可忽视的调度开销

---

## 3. 架构设计

### 3.1 系统总览

```
┌─────────────────────────────────────────────────────────────┐
│                        JANUS 芯片                            │
│                                                             │
│  ┌──────────────────┐         ┌──────────────────────────┐  │
│  │   P-Golem 阵列   │         │    D-Golem 阵列           │  │
│  │  （N 个实例）     │         │   （M 个实例）             │  │
│  │                  │         │                          │  │
│  │  ┌────────────┐  │         │  ┌────────────────────┐  │  │
│  │  │ RV 核      │  │         │  │ RV 核（Vanadis）    │  │  │
│  │  │（Vanadis） │  │         │  │ KV 页管理器         │  │  │
│  │  └────────────┘  │         │  └────────────────────┘  │  │
│  │  ┌────────────┐  │         │  ┌────────────────────┐  │  │
│  │  │ CIM 阵列   │  │         │  │ CIM 阵列            │  │  │
│  │  │（GEMM 优化）│  │         │  │（GEMV 优化）         │  │  │
│  │  └────────────┘  │         │  └────────────────────┘  │  │
│  │  ┌────────────┐  │         │  ┌────────────────────┐  │  │
│  │  │ SRAM       │  │         │  │ SRAM（KV 页存储）   │  │  │
│  │  └────────────┘  │         │  └────────────────────┘  │  │
│  └────────┬─────────┘         └────────────┬─────────────┘  │
│           │                                │                 │
│           └──────────┬─────────────────────┘                 │
│                      │                                       │
│              ┌───────┴────────┐                              │
│              │  KV 迁移总线   │                              │
│              └────────────────┘                              │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 P-Golem 设计

- **RV 核职责**：序列切分、负载均衡、GEMM 触发
- **CIM 阵列配置**：大 Tile 高并行度，最大化矩阵乘法吞吐量
- **自定义指令**：
  - `golem.p.gemm rd, rs1, rs2` —— 触发 CIM GEMM 计算
  - `golem.p.split rd, rs1, n` —— 将输入序列切分为 n 块
  - `golem.p.sync` —— 跨 P-Golem 实例同步

### 3.3 D-Golem 设计

- **RV 核职责**：KV 页分配/释放、GEMV 触发、预取控制
- **SRAM 布局**：固定大小 KV 页（参考 PagedAttention），硬件自管理
- **自定义指令**：
  - `golem.kv.alloc rd, size` —— 分配 KV Cache 页
  - `golem.kv.free rs1` —— 释放 KV Cache 页
  - `golem.kv.attn rd, rs1, rs2` —— 触发带 KV 查找的 Attention GEMV
  - `golem.kv.migrate dst, src, size` —— 发起 KV 数据迁移至另一 D-Golem

### 3.4 KV 迁移总线

- 专用片上总线，连接 P-Golem 输出与 D-Golem KV SRAM
- 迁移由源端 D-Golem RV 核发起，无需中央控制器
- Prefill 完成后，P-Golem 通过此总线将 KV Cache 推送至指定 D-Golem

### 3.5 与现有方案对比

| 特性 | 传统 CIM 加速器 | DistServe/Splitwise（GPU） | **JANUS（本方案）** |
|------|---------------|--------------------------|-------------------|
| PD 分离粒度 | ❌ 混合运行 | ✅ 节点级 | ✅ **片上级** |
| KV 管理方式 | 软件 | 软件（vLLM） | ✅ **硬件** |
| 调度方式 | 中心化 | 中心化 | ✅ **去中心化** |
| KV 迁移延迟 | 无 | PCIe/NVLink（微秒~毫秒） | ✅ **片上总线（纳秒）** |
| RISC-V 控制 | ❌ | ❌ | ✅ **每 Golem 独立 RV 核** |

---

## 4. 相关工作与参考文献

### 4.1 核心必读文献

| 序号 | 论文 | 会议/期刊 | 年份 | 与本研究的关系 |
|------|------|---------|------|-------------|
| 1 | **Splitwise**：Efficient Generative LLM Inference Using Phase Splitting — Patel et al. | ISCA | 2024 | 直接前驱：硬件异构化 PD 分离 |
| 2 | **DistServe**：Disaggregating Prefill and Decoding for Goodput-optimized LLM Serving — Zhong et al. | OSDI | 2024 | 系统级 PD 分离基线 |
| 3 | **vLLM/PagedAttention**：Efficient Memory Management for LLM Serving — Kwon et al. | SOSP | 2023 | KV Cache 管理基础方案 |
| 4 | **Mooncake**：A KVCache-centric Disaggregated Architecture for LLM Serving — Qin et al. | arXiv | 2024 | 工业界 KV 中心化分离最新实践 |
| 5 | **Sarathi-Serve**：Efficient LLM Inference by Piggybacking Decodes with Chunked Prefills — Agrawal et al. | OSDI | 2024 | Chunked Prefill 对比基线 |
| 6 | **Efficiently Scaling Transformer Inference** — Pope et al. | MLSys | 2023 | Prefill/Decode 计算特性分析基础 |

### 4.2 文献叙事逻辑（用于论文 Related Work 章节）

```
背景引入：
  vLLM/PagedAttention (SOSP'23)
      └─► 解决了 KV Cache 碎片化问题，但调度仍在软件层

问题升级：
  Sarathi-Serve (OSDI'24)
      └─► Chunked Prefill 缓解了 PD 干扰，但根本矛盾未解决

系统级解决方案：
  DistServe (OSDI'24) + Splitwise (ISCA'24) + Mooncake (2024)
      └─► PD 分离在数据中心级被验证有效，但依赖 PCIe/NVLink，延迟高

本文贡献：
  JANUS
      └─► 将 PD 分离下沉至片上 CIM 阵列，通过 RISC-V 核实现硬件级去中心化调度
```

---

## 5. 仿真评估计划

### 5.1 仿真工具

- **框架**：SST（Structural Simulation Toolkit）
- **处理器模型**：Vanadis（SST 内置 RISC-V 乱序核模型）
- **CIM 阵列模型**：自定义 SST 组件，建模 GEMM/GEMV 时序

### 5.2 SST 仿真组件规划

```
SST 顶层
├── P-Golem 组件 × N
│   ├── RV 核模型（Vanadis）
│   ├── CIM 阵列模型（GEMM 时序）
│   └── 序列切分器
├── D-Golem 组件 × M
│   ├── RV 核模型（Vanadis）
│   ├── CIM 阵列模型（GEMV 时序）
│   └── KV 页管理器
├── KV 迁移总线
└── 分布式请求调度器
```

### 5.3 评估指标

| 指标 | 说明 | 对比基线 |
|------|------|---------|
| **TTFT** | 首 Token 延迟 | 同构 Golem 阵列 |
| **TPOT** | 每 Token 生成时间 | 同构 Golem 阵列 |
| **KV 管理开销** | 页分配/释放延迟 | 软件 PagedAttention |
| **资源利用率** | P/D Golem 占用率 | 混合调度方案 |
| **能效比** | TOPS/W | GPU 基线 |
| **KV 迁移延迟** | P→D 传输时间 | PCIe/NVLink |

### 5.4 测试负载

- **模型**：LLaMA-7B / LLaMA-13B
- **数据集**：ShareGPT（真实对话长度分布）
- **请求模式**：泊松到达过程，模拟在线服务场景

---


## 6. 待解决问题与 TODO

### 6.1 研究层面待回答问题

- [ ] P-Golem 与 D-Golem 的最优数量比（N:M）如何确定？
- [ ] KV 迁移总线的带宽需求如何量化？（取决于模型大小与 Batch Size）
- [ ] 去中心化调度是否会引入新的负载不均衡问题？
- [ ] 与 GPU 基线相比，TTFT 和 TPOT 的预期改善幅度是多少？

### 6.2 工程层面 TODO

- [ ] 完成 P-Golem SST 组件建模（目标：2026年7月）
- [ ] 完成 D-Golem SST 组件建模（目标：2026年7月）
- [ ] 实现 KV 迁移总线 SST 组件（目标：2026年8月）
- [ ] 跑通 LLaMA-7B 端到端仿真（目标：2026年8月）
- [ ] 完成 HPCA 2027 论文初稿（目标：2026年9月）

### 6.3 当前进展

- 仿真阶段：SST 框架搭建中
- 架构阶段：P/D-Golem 微架构设计完成（概念级）
- 指令集：自定义 RISC-V 扩展指令初步定义完成
- 文献调研：核心 PD 分离文献调研完成

---

## 7. 项目命名说明

**选定名称**：JANUS

**命名来源**：罗马双面神 Janus，一张脸望向过去（处理输入 = P-Golem），一张脸望向未来（生成输出 = D-Golem），完美对应本架构的双核异构设计哲学。

**论文标题示例**：
> *JANUS: A Disaggregated Prefill-Decode CIM Manycore Accelerator for LLM Inference with Heterogeneous RISC-V Golem Arrays*

---

*文档结束 | 如需更新请联系项目负责人*
