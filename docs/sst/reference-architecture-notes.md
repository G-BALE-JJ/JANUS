# SST 参考架构记录

## 目标

本文档记录从 `/data4/jjgong/RISC-V-CIM-Manycore-SST` 观察到的工程事实，作为 JANUS 后续接入真实 SST/Golem 框架的参考索引。

## 参考仓库

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST
```

## Golem element 入口

```text
src/sst/elements/golem/golem.cc
src/sst/elements/golem/README.md
```

`golem.cc` 注册/包含的主要模块包括：

- `rocc/roccAnalog.h`
- `rocc/roccAnalogFloat.h`
- `rocc/roccAnalogInt.h`
- `array/computeArray.h`
- `array/mvmComputeArray.h`
- `array/mvmFloatArray.h`
- `array/mvmIntArray.h`
- `groupctrl/groupctrl.h`
- `requestscheduler/requestscheduler.h`

## Topology builder

重要参考路径：

```text
src/sst/elements/golem/tests/architecture/noc_builder.py
src/sst/elements/golem/tests/architecture/cpu_builder.py
src/sst/elements/golem/tests/architecture/ncores_selfcom_dma_ctrl.py
```

观察：

- `noc_builder.py` 封装 Merlin mesh router 构建和 local port attach。
- `cpu_builder.py` 集中处理 Vanadis CPU、RoCC、array、global memory、scheduler 等环境参数。
- `ncores_selfcom_dma_ctrl.py` 负责将 CPU rows、memory rows、NoC routers、HBM/data nodes 等组成完整 SST topology。

## Small tests

重要参考路径：

```text
src/sst/elements/golem/tests/small/mvm_noc_int_array
src/sst/elements/golem/tests/small/mvm_noc_softmax_cpu
src/sst/elements/golem/tests/small/lenet5
```

观察：

- `mvm_noc_int_array` 更接近 GEMM/MVM runtime。
- `mvm_noc_softmax_cpu` 展示了在 Golem runtime 旁引入 CPU fallback/operator API 的方式。
- `lenet5` 展示了多算子/多阶段 workload 的组织方式，但对 JANUS 第一阶段可能偏重。

## Config 与运行脚本

重要参考路径：

```text
src/sst/elements/golem/tests/configs/
src/sst/elements/golem/tests/run_noc_dma_pipeline.sh
```

观察：

- configs 使用多个 `.env` 文件拆分核心 GEMM、DMA、network、latency、debug、run 参数。
- `run_noc_dma_pipeline.sh` 是现有端到端管线入口。
- JANUS 当前不包装或运行这些脚本，只记录作为后续接入参考。

## 当前建议

JANUS 后续如果进入实现，优先顺序应为：

1. 复用现有 topology builder 思路。
2. 复用现有 small runtime 组织方式。
3. 复用 request scheduler / groupctrl / workercmdproc 的边界。
4. 最后再决定是否新增 JANUS-specific SST element。

