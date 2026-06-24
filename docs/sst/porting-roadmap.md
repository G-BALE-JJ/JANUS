# SST Porting Roadmap

## 目标

本文档给出 JANUS 从当前文档骨架走向真实 SST/Golem 框架的迁移路线。当前不执行迁移、不跑仿真。

## Roadmap

### Step 0: 当前文档骨架

当前状态：

- JANUS 只有架构文档和参考映射。
- 不包含真实实现。
- 不依赖 SST 环境。

完成条件：

- 用户确认项目边界。
- `docs/architecture` 和 `docs/sst` 能解释后续实现路径。

### Step 1: 选择参考 small case

候选：

```text
RISC-V-CIM-Manycore-SST/src/sst/elements/golem/tests/small/mvm_noc_int_array
RISC-V-CIM-Manycore-SST/src/sst/elements/golem/tests/small/mvm_noc_softmax_cpu
```

建议：

- 若先做 P-Golem，选择 `mvm_noc_int_array`。
- 若先做 D-Golem/KV path，选择 `mvm_noc_softmax_cpu` 的 CPU/operator fallback 组织方式作为控制路径参考。

### Step 2: 定义 JANUS descriptor

产物：

- P-Golem prefill descriptor。
- D-Golem decode descriptor。
- KV migration descriptor。

要求：

- 字段必须能映射到现有 `WorkerTaskListHeader`、`WorkerWindowDescriptor`、`RequestSchedulerMsg` 或说明为什么不能。

### Step 3: 编写 JANUS small case 计划

产物：

- 一个 small case 设计文档。
- 明确输入/输出。
- 明确是否修改参考仓库。
- 明确 build/run 命令，但不在计划阶段运行。

### Step 4: 最小实现

候选实现方式：

1. 在参考仓库中增加一个 small runtime case。
2. 在 JANUS 中写 wrapper 文档和脚本，调用参考仓库。
3. 在 Golem component 中增加 role/config 扩展。

推荐顺序：

```text
small case -> descriptor -> wrapper -> component extension
```

### Step 5: Smoke test

目标：

- 能够构造一个最小 P->D 数据流。
- 能够产生日志或 stats 证明 P-Golem task、KV migration、D-Golem task 的顺序。

当前不执行。

## 风险控制

| 风险 | 控制方式 |
|---|---|
| 直接陷入 SST 环境问题 | 在实现前先写 descriptor 和 small case plan |
| 修改范围过大 | 先新增 small case，不先改核心 element |
| P/D-Golem 边界不清 | 通过 `golem-reference-map.md` 和 component boundary plan 固化 |
| KV path 语义过大 | 第一版只做 KV state/block，不做完整 PagedAttention |

