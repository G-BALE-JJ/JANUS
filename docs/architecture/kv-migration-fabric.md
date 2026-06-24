# KV Migration Fabric

## 目标

KV migration fabric 表示 JANUS 中 P-Golem 到 D-Golem 的 KV state 迁移路径。当前阶段不新增真实 fabric component，而是优先参考现有 Golem/SST 的 NoC 和 DMA 组织。

## 非目标

- 不实现专用总线。
- 不定义 NoC cycle-level 参数。
- 不运行带宽 sweep。
- 不承诺纳秒级延迟。

## 参考路径

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem/tests/architecture/noc_builder.py
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem/tests/architecture/ncores_selfcom_dma_ctrl.py
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem/requestscheduler/
```

## 工程解释

第一版迁移路径建议表达为：

```text
P-Golem worker
  -> submit KV transfer request
  -> request scheduler
  -> NoC/DMA transfer
  -> D-Golem local memory endpoint
  -> completion event
```

这样可以先复用现有 scheduler、credit、NoC router 和 memory endpoint 思路。只有当现有路径无法表达 P->D 专用语义时，再新增 `janus_kv_fabric`。

## 候选 descriptor

```text
KVMigrationDescriptor {
  request_id
  layer_id
  src_golem_id
  dst_golem_id
  src_addr
  dst_addr
  bytes
  priority
  completion_flag_addr
}
```

字段名只是第一版草案。后续需要与 `RequestSchedulerMsg` 的字段对齐：

```text
requestId
srcAddr
dstAddr
bytes
targetNode
completionFlagAddr
completionValue
```

## 后续判断标准

先尝试用现有 request scheduler 表达 KV migration。如果出现以下问题，再考虑专用 fabric：

- 现有 request scheduler 无法区分 P->D KV traffic 和普通 tensor traffic。
- KV transfer 需要独立 QoS/priority。
- KV transfer 需要与 decode scheduling 形成强依赖。
- D-Golem local memory 需要专门的 admission/placement protocol。

## 未决问题

- P-Golem 到 D-Golem 是否一对一绑定。
- KV migration 是否需要 multicast。
- 是否需要迁移完成后自动触发 decode。
- KV traffic 是否应当使用独立 virtual network。

