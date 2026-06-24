# Control Plane

## 目标

本文档定义 JANUS 控制面的工程分层，并说明它如何参考现有 Golem/SST 的 `groupctrl/`、`requestscheduler/` 和 `workercmdproc/`。

## 非目标

- 不设计完整 serving scheduler。
- 不实现中心化调度器。
- 不定义所有控制消息编码。
- 不运行调度实验。

## 分层

JANUS 控制面建议分为三层：

```text
Host/runtime level
  -> 接收外部请求，生成 P/D task descriptor

Group/request scheduler level
  -> 分配 worker、本地仲裁、数据搬运调度

Worker command processor level
  -> 每个 P/D-Golem 内部推进任务状态
```

## 参考组件

### `groupctrl/`

适合参考 group-local 控制语义：

```text
REQUEST
GRANT
DONE
FINISHED
GROUP_DONE
```

JANUS 的去中心化调度不应一开始落成单一全局中心调度器。更合理的路线是先复用 group-local manager/worker 结构。

### `requestscheduler/`

适合参考数据搬运调度语义：

```text
SUBMIT
DONE
node credit
batch submit
batch done
```

JANUS 的 KV migration 应优先尝试映射成 request scheduler transaction。

### `workercmdproc/`

适合参考 worker-local command execution：

```text
startWindow()
tick()
handleArrayDone()
isBusy()
```

P-Golem 和 D-Golem 都需要本地任务推进逻辑。第一版应优先从 worker command processor 的状态机思想出发。

## JANUS 控制事件草案

```text
PrefillTaskSubmit
PrefillTaskDone
KVTransferSubmit
KVTransferDone
DecodeTaskSubmit
DecodeTaskDone
RequestDone
```

这些事件当前只是工程语义，不是 SST event class 定义。

## 未决问题

- Host/runtime 是否直接看到 KV migration。
- P-Golem 与 D-Golem 是否共享同一个 group manager。
- Decode task 是否必须等待 KVTransferDone。
- RequestScheduler 是否需要扩展 role 字段区分 prefill/decode。

