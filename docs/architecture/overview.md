# JANUS 架构总览

## 目标

本文档定义 JANUS 当前阶段的工程架构视图。它不描述论文贡献，也不承诺性能结论；它只回答一个问题：如果未来要把 JANUS 落到 SST/Golem 风格的硬件仿真框架中，应该有哪些核心组件边界。

## 非目标

- 不定义完整 workload 格式。
- 不定义真实 ISA 编码。
- 不定义论文实验矩阵。
- 不实现 SST component。
- 不运行仿真。

## 顶层视图

JANUS 的核心是一个面向 LLM 推理的异构 Golem 阵列：

```text
Host / runtime
  |
  v
Control plane
  |
  +--> P-Golem array: prefill-oriented GEMM execution
  |
  +--> KV migration fabric: P->D KV state movement
  |
  +--> D-Golem array: decode-oriented GEMV execution and KV resident data
```

当前工程上建议先把 JANUS 映射到现有 Golem/SST 的组件边界：

```text
P-Golem
  -> array/
  -> rocc/
  -> workercmdproc/

D-Golem
  -> array/
  -> globalmemory/
  -> requestscheduler/
  -> workercmdproc/

Control plane
  -> groupctrl/
  -> requestscheduler/

KV migration fabric
  -> tests/architecture/noc_builder.py
  -> existing DMA / NoC path
```

## 组件边界

### P-Golem

P-Golem 负责 prefill 阶段的大块矩阵计算。当前不要求它立即成为一个独立 SST element；更保守的路线是先把它作为现有 Golem compute array + worker command processor 的配置化角色。

关键职责：

- 接收 prefill GEMM task。
- 执行 prompt/token block 的矩阵乘。
- 产生供 D-Golem 使用的 KV state。
- 将 KV write/migration descriptor 交给控制面或迁移路径。

### D-Golem

D-Golem 负责 decode 阶段的 GEMV/attention-like 访问，以及 KV cache resident data 的本地管理。当前建议优先参考 `globalmemory/` 和 `requestscheduler/`，不要先发明独立的大型 KV 子系统。

关键职责：

- 接收来自 P-Golem 的 KV state。
- 在本地 memory service 中维护 KV resident data。
- 服务 decode GEMV / attention read path。
- 通过本地控制面处理 KV page/state 生命周期。

### KV migration fabric

KV migration fabric 不是第一阶段就新增的硬件模块。当前建议先把它理解为一个使用现有 NoC/DMA path 的数据通路：

```text
P-Golem local output
-> migration descriptor
-> NoC / DMA transfer
-> D-Golem local memory
```

后续如果证明现有 NoC/DMA path 无法表达 JANUS 的需求，再考虑新增专用 fabric component。

### Control plane

控制面拆成三层：

```text
Host/runtime level
-> group/request scheduler level
-> worker command processor level
```

其中：

- `groupctrl/` 更接近 group-local grant/done/finished 控制。
- `requestscheduler/` 更接近数据搬运和请求调度。
- `workercmdproc/` 更接近每个 worker 本地的任务推进状态机。

## 后续实现任务

1. 固化 P-Golem 和 D-Golem 的最小 descriptor。
2. 找出 `workercmdproc/` 中哪些字段可以直接复用，哪些字段需要为 KV path 扩展。
3. 找出 `requestscheduler/` 是否能表达 P->D migration。
4. 在不修改参考仓库的前提下，先形成 JANUS 的 component boundary plan。

## 未决问题

- P-Golem 是否应当拥有独立的 prefill scheduler。
- D-Golem 的 KV resident data 是否由 `globalmemory/` 扩展即可。
- P->D migration 是数据面操作还是控制面可见事件。
- JANUS 是否最终 fork Golem element，还是作为 Golem element 的新配置/新 subcomponent。

