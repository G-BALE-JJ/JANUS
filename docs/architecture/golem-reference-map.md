# Golem/SST 参考映射

## 目标

本文档记录 JANUS 概念与 `/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem` 现有组件之间的参考关系。它是后续实现前的边界地图，避免 JANUS 在空目录里重复设计已有能力。

## 参考路径

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem
```

## 顶层组件观察

| Golem 路径 | 当前含义 | JANUS 参考方式 |
|---|---|---|
| `array/` | Compute array / MVM array 抽象 | P-Golem/D-Golem 的 CIM array 基础 |
| `rocc/` | RoCC coprocessor interface | RISC-V/Vanadis 与 array 连接方式 |
| `globalmemory/` | Golem memory abstraction | D-Golem KV local memory 候选基础 |
| `groupctrl/` | group-local control endpoint | 去中心化控制面候选基础 |
| `requestscheduler/` | transfer request scheduler endpoint | KV/data migration scheduling 候选基础 |
| `workercmdproc/` | worker command processor | P/D-Golem 本地任务状态机候选基础 |
| `tests/architecture/` | SST topology builder | JANUS 拓扑和 NoC 构建参考 |
| `tests/small/` | small runtime/test cases | JANUS 后续 smoke case 组织参考 |

## JANUS 到 Golem 的概念映射

| JANUS 概念 | 建议参考组件 | 说明 |
|---|---|---|
| P-Golem compute tile | `array/`, `rocc/`, `workercmdproc/` | 先作为 GEMM-oriented worker 角色，不急于新增 SST element |
| D-Golem compute tile | `array/`, `globalmemory/`, `workercmdproc/` | 先作为 GEMV/KV-oriented worker 角色 |
| KV resident storage | `globalmemory/` | 优先评估扩展现有 memory abstraction |
| P->D KV transfer | `requestscheduler/`, `tests/architecture/noc_builder.py` | 优先复用 request scheduler + NoC/DMA path |
| local control | `groupctrl/` | 参考 REQUEST/GRANT/DONE/FINISHED/GROUP_DONE 的控制语义 |
| worker-side execution | `workercmdproc/` | 参考 task header、window descriptor、tick/array done 状态推进 |
| topology | `tests/architecture/ncores_selfcom_dma_ctrl.py` | 参考 CPU row、memory row、router placement、Merlin mesh 构建方式 |

## 具体文件参考

### `array/`

现有 array 抽象包括 int/float MVM array。JANUS 需要判断：

- P-Golem 是否需要 GEMM-specific tiling wrapper。
- D-Golem 是否只需要 GEMV/MVM array，还是需要 KV-aware read path。
- array latency、pipeline depth、MAC rate 是否继续使用环境变量配置。

### `rocc/`

RoCC 接口是 Golem 连接 Vanadis/RISC-V core 的关键路径。JANUS 当前不定义真实 ISA 编码，但后续若要让 P/D-Golem 由 RV core 触发，应先参考 RoCC 路径。

### `globalmemory/`

D-Golem 的 KV local memory 不应一开始独立发明。建议先评估：

- 是否能在 `globalmemory/` 上表达 KV block/page。
- 是否能记录 D-Golem local address window。
- 是否能支持 decode read path 所需的 gather-like access。

### `groupctrl/`

`groupctrl/` 已经有 worker/manager role，以及 REQUEST、GRANT、DONE、FINISHED、GROUP_DONE 消息。JANUS 的去中心化调度应优先映射到这些语义，而不是先新增中心调度器。

### `requestscheduler/`

`requestscheduler/` 已经包含 transfer request、node credit、batch submit/done 等机制。JANUS 的 P->D KV migration 可以先被建模为特殊的数据搬运请求，再判断是否需要新 message type。

### `workercmdproc/`

`workercmdproc/` 已经有 worker task list header、window descriptor 和 tick-based progression。JANUS 的 P-Golem prefill 和 D-Golem decode 都可以先尝试映射成 worker-local command processor 的不同 task role。

## 当前建议

短期不 fork 大量代码。先用文档明确：

1. 哪些 JANUS 概念能直接复用 Golem 组件。
2. 哪些需要新增 descriptor 字段。
3. 哪些必须新增 subcomponent 或 element。
4. 哪些只是 topology/test runtime 变化。

## 未决问题

- JANUS 是否需要在 Golem element 中新增 role 参数，例如 `role=prefill` / `role=decode`。
- KV transfer 是否只是 request scheduler 的新 transaction type。
- KV resident data 是否需要 page table，还是第一版只建模 contiguous local memory。
- D-Golem 是否需要专门的 completion/emit token event。

