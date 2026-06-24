# KV Cache Path

## 目标

本文档定义 JANUS 当前阶段对 KV cache path 的工程理解。当前不实现完整 PagedAttention，也不定义软件 runtime API；重点是明确 KV state 从 P-Golem 到 D-Golem 的硬件数据路径。

## 非目标

- 不实现 block table。
- 不实现 copy-on-write。
- 不实现 prefix sharing。
- 不定义软件 serving scheduler。
- 不承诺硬件完全等价于 vLLM PagedAttention。

## 初始路径

第一版 KV path 建议按数据通路理解：

```text
P-Golem prefill output
  -> KV write descriptor
  -> P-Golem local/staging memory
  -> KV migration request
  -> NoC / DMA path
  -> D-Golem local memory
  -> Decode GEMV / attention read path
```

## 与 Golem 组件的关系

| 路径阶段 | 候选参考组件 |
|---|---|
| P-Golem output staging | `globalmemory/` 或 worker local buffer |
| migration request | `requestscheduler/` |
| NoC transfer | `tests/architecture/noc_builder.py` |
| D-Golem local storage | `globalmemory/` |
| decode read | `workercmdproc/` + `array/` |

## 第一版 KV state 粒度

建议第一版不要直接叫 page table，而使用更中性的 `KV block/state`：

```text
KVState {
  request_id
  layer_id
  token_begin
  token_count
  src_addr
  dst_addr
  bytes
  src_golem_id
  dst_golem_id
}
```

后续如果需要支持真实 paging，再把 `KVState` 拆成 logical block、physical page、refcount、free list 等结构。

## 风险

- 如果过早声称硬件级 PagedAttention，后续实现会被完整软件语义牵制。
- 如果不先复用 `globalmemory/`，可能会重复实现 memory window、DMA 和 address mapping。
- 如果 P->D 迁移没有进入 scheduler/NoC 口径，后续性能模型会缺少拥塞和仲裁边界。

## 未决问题

- KV block 是否固定大小。
- KV metadata 放在 host/runtime、D-Golem local memory，还是 SST component 内部状态。
- KV migration completion 如何通知 decode task。
- D-Golem 是否允许 KV eviction/free，还是第一版只做 append-only。

