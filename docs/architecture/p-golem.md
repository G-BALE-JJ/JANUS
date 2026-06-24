# P-Golem

## 目标

P-Golem 是 JANUS 中面向 prefill 阶段的 Golem 角色。当前文档只定义工程职责和与现有 Golem/SST 组件的参考关系，不实现真实 SST component。

## 非目标

- 不定义完整 ISA。
- 不实现 GEMM kernel。
- 不实现真实 prefill runtime。
- 不承诺 tile shape 或性能数字。

## 工程职责

P-Golem 的第一版职责建议保持简单：

1. 接收 prefill GEMM task。
2. 驱动本地 CIM/MVM array 执行矩阵计算。
3. 生成 KV state 或 KV write descriptor。
4. 将 KV state 交给 P->D migration path。

## 参考 Golem 组件

| 参考组件 | 参考点 |
|---|---|
| `array/` | P-Golem 的计算阵列基础 |
| `rocc/` | RV/Vanadis 触发 array 的接口模式 |
| `workercmdproc/` | 本地 worker command state machine |
| `requestscheduler/` | 数据读取、预取、迁移请求 |
| `tests/small/mvm_noc_int_array` | GEMM/MVM small case 组织方式 |

## 初始设计假设

- P-Golem 不必一开始作为独立 SST element。
- P-Golem 可以先作为现有 Golem worker 的 prefill role。
- P-Golem 的输出不直接绑定 D-Golem 实现细节，而是输出 KV migration descriptor。
- P-Golem 的 scheduler 不应先做成中心化全局调度器；优先参考 `groupctrl/` 和 `requestscheduler/` 的局部控制方式。

## 后续需要定义的 descriptor

P-Golem prefill descriptor 至少需要包含：

```text
request_id
layer_id
token_begin
token_count
input_base_addr
weight_base_addr
kv_output_base_addr
gemm_m
gemm_n
gemm_k
dtype
target_d_golem_id
```

字段名目前只是草案，后续必须对照 `WorkerTaskListHeader` 和 `WorkerWindowDescriptor` 再收敛。

## 未决问题

- P-Golem 的 GEMM 是否直接映射到现有 MVM array 多次调用。
- P-Golem 是否需要跨 tile synchronization。
- P-Golem 到 D-Golem 的目标选择由谁决定。
- P-Golem 是否需要本地 KV staging buffer。

