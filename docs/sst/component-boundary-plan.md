# SST Component Boundary Plan

## 目标

本文档定义 JANUS 后续进入真实 SST/Golem 实现时的组件边界推进顺序。当前不实现代码，只记录设计路线。

## 基本原则

- 先复用现有 Golem 组件边界，再新增 JANUS-specific component。
- 先扩展 descriptor 和 runtime small case，再修改底层 SST element。
- 先让 P-Golem/D-Golem 作为 role 出现，再决定是否拆成独立 element。
- 任何新增组件都必须能解释为什么 `array/`、`globalmemory/`、`requestscheduler/`、`groupctrl/`、`workercmdproc/` 无法承担。

## 阶段计划

### Stage 1: 文档映射

状态：当前阶段。

目标：

- 固化 JANUS 概念到 Golem 组件边界的映射。
- 明确不需要立即新增 SST element。
- 明确后续第一个 small case 的候选路径。

### Stage 2: Runtime descriptor 草案

目标：

- 定义 P-Golem prefill descriptor。
- 定义 D-Golem decode/KV descriptor。
- 定义 KVMigrationDescriptor。
- 对照 `WorkerTaskListHeader`、`WorkerWindowDescriptor`、`RequestSchedulerMsg` 检查字段。

### Stage 3: Small case 设计

目标：

- 选择 `mvm_noc_int_array` 作为 prefill/GEMM 参考，或选择 `mvm_noc_softmax_cpu` 作为 mixed compute/control 参考。
- 设计一个不修改大框架的 JANUS small case。
- 明确该 small case 的输入、输出和完成判据。

### Stage 4: Topology extension

目标：

- 参考 `ncores_selfcom_dma_ctrl.py`，定义 P-Golem row、D-Golem row、memory row 的 placement。
- 评估是否使用同一 Merlin mesh。
- 评估是否需要独立 local ports 或 virtual networks。

### Stage 5: Component extension

目标：

- 如果 role/config 不足，再新增 JANUS-specific subcomponent。
- 可能新增的最小对象包括：
  - `JanusKVMetadata`
  - `JanusKVMigrationDescriptor`
  - `JanusDecodeWorkerRole`
  - `JanusPrefillWorkerRole`

### Stage 6: 独立 SST element 决策

只有在以下条件成立时才考虑独立 element：

- 现有 Golem array/RoCC/globalmemory/request scheduler 无法通过配置或小扩展表达 P/D-Golem。
- KV path 需要专用端口或事件类型。
- P-Golem/D-Golem 的生命周期和现有 worker model 根本不同。

## 当前不建议的做法

- 直接复制整个 `RISC-V-CIM-Manycore-SST` 到 JANUS。
- 直接新增 `janus` SST element 大文件。
- 在没有 descriptor 的情况下修改 `workercmdproc/`。
- 在没有 small case 的情况下修改 topology builder。
- 先写 workload/schema 再找硬件路径。

## 未决问题

- JANUS 是否应作为 Golem 的 feature branch 发展，还是最终独立仓库。
- 第一个实现应从 P-Golem prefill 还是 D-Golem KV path 开始。
- P/D-Golem role 是否能通过环境变量表达。
- KV path 的 metadata 是否进入 SST event，还是仅 runtime memory layout。

