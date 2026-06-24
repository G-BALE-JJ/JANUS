# D-Golem

## 目标

D-Golem 是 JANUS 中面向 decode 阶段的 Golem 角色。它负责 decode GEMV/attention-like 计算，并维护本地 KV resident data。当前文档只定义工程边界。

## 非目标

- 不实现 PagedAttention。
- 不定义完整 KV page table 数据结构。
- 不实现真实 decode runtime。
- 不运行 SST。

## 工程职责

D-Golem 的第一版职责建议包括：

1. 接收 P-Golem 迁移来的 KV state。
2. 将 KV state 放入 D-Golem local memory。
3. 接收 decode task。
4. 驱动本地 array 执行 GEMV/MVM。
5. 输出 token-level completion event 或 decode done event。

## 参考 Golem 组件

| 参考组件 | 参考点 |
|---|---|
| `array/` | decode GEMV/MVM 执行基础 |
| `globalmemory/` | KV resident data 的候选存储抽象 |
| `requestscheduler/` | KV/data transfer 请求调度 |
| `workercmdproc/` | decode worker command processor |
| `groupctrl/` | D-Golem local control / manager-worker 协作 |

## 初始设计假设

- D-Golem 不先实现完整软件 PagedAttention 等价语义。
- 第一版只要求表达 KV state 的写入、驻留和读取路径。
- KV local memory 优先参考 `globalmemory/`，不要先新增完全独立 memory subsystem。
- Decode task 可以先被表达为 GEMV/MVM-like worker task，再逐步加入 KV metadata。

## 后续需要定义的 descriptor

D-Golem decode/KV descriptor 至少需要包含：

```text
request_id
layer_id
token_index
kv_base_addr
kv_bytes
query_base_addr
weight_base_addr
output_base_addr
decode_m
decode_n
decode_k
dtype
```

KV resident data descriptor 至少需要包含：

```text
request_id
layer_id
kv_block_id
local_addr
bytes
valid_token_begin
valid_token_count
owner_d_golem_id
```

## 未决问题

- KV resident data 是否第一版就需要 page/block abstraction。
- D-Golem 是否需要多个 local memory bank。
- Decode 的 attention read path 是否能被现有 MVM array + global memory 表达。
- D-Golem 的 token emit event 应该由 worker command processor 还是 host/runtime 观察。

