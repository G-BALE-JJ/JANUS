# JANUS 发现与决策摘要

## 当前事实

- `JANUS` 当前是一个新项目目录，原始内容只有 `idea.md` 和 Git 元数据。
- 用户当前不要求做学术论文工作。
- 用户当前不要求跑通验证或运行端到端仿真。
- 用户不希望创建 `docs/simulation`、`docs/isa`、`docs/workloads`、`docs/development`、`schemas` 等泛化目录。
- JANUS 当前应参考 `/data4/jjgong/RISC-V-CIM-Manycore-SST` 的真实 Golem/SST 架构，而不是先凭空设计完整仿真软件栈。

## 参考仓库观察

主要参考路径：

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem
```

重要组件边界：

| 路径 | 作用 | JANUS 参考意义 |
|---|---|---|
| `array/` | Compute array / MVM array 抽象 | P-Golem 和 D-Golem 的 CIM array 基础 |
| `rocc/` | RoCC analog coprocessor interface | 与 Vanadis/RISC-V 核连接的参考接口 |
| `globalmemory/` | Golem memory abstraction | D-Golem KV local memory 的候选基础 |
| `groupctrl/` | group-local control endpoint | 去中心化控制面的候选基础 |
| `requestscheduler/` | transfer request scheduler endpoint | KV/data migration 调度的候选基础 |
| `workercmdproc/` | worker command processor | P/D-Golem 本地任务执行状态机的候选基础 |
| `tests/architecture/` | SST topology builder | JANUS 拓扑搭建的候选参考 |
| `tests/small/` | small runtime/test cases | JANUS smoke case 组织方式参考 |

## 当前决策

| 决策 | 原因 |
|---|---|
| 保持 JANUS 骨架薄 | 当前还没有真实 JANUS SST 框架 |
| 只创建 `docs/architecture` 和 `docs/sst` | 贴近硬件组件边界，不制造多余抽象 |
| 暂不创建 schema/ISA/workload 文档 | 用户明确不需要这些目录 |
| 暂不运行仿真 | 用户明确不需要跑通验证 |
| 暂不修改参考仓库 | 先建立 JANUS 自身文档和映射 |
| 后续优先复用 Golem 组件边界 | 降低重新发明 SST element 的风险 |

## 待确认问题

1. P-Golem 是否作为现有 Golem `array/rocc/workercmdproc` 的配置化变体，还是作为新 SST element。
2. D-Golem 的 KV SRAM/local memory 是否扩展 `globalmemory/`。
3. P->D KV migration 是否复用现有 Merlin mesh + DMA path。
4. 是否需要在 JANUS 中维护一份参考 Golem 代码摘要，还是只维护路径索引。
5. 第一个 JANUS small case 应该从 GEMM prefill 开始，还是从 decode/GEMV + KV path 开始。

