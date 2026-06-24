# JANUS-local Golem SST Element

## 目标

本目录是 JANUS 项目内的 Golem SST element 源码区。它从 `/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem` 导入，用于学习、裁剪和自定义修改 Golem 组件。

## 当前策略

- 在 JANUS 内修改 `sst/elements/golem/`。
- 不直接修改 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。
- 不维护过早拆分的 `sst/elements/janus/`。
- 保留 upstream copyright/header。
- 排除构建产物、缓存、日志和 runtime artifacts。

## 导入范围

详见 [IMPORT_NOTES.md](IMPORT_NOTES.md)。

## 主要子目录

| 路径 | 作用 |
|---|---|
| `array/` | compute/MVM array 抽象 |
| `rocc/` | RoCC analog coprocessor interface |
| `globalmemory/` | Golem memory abstraction |
| `groupctrl/` | group-local control endpoint |
| `requestscheduler/` | transfer request scheduler endpoint |
| `workercmdproc/` | worker command processor prototype |
| `tests/architecture/` | topology/CPU/NoC builder 参考 |
| `tests/configs/` | Golem test env 配置参考 |
| `tests/small/` | small runtime/test case 参考 |

## 下一步建议

第一阶段不要急于改动所有模块。建议先完成：

1. 阅读 `golem.cc`，理解 element 注册入口。
2. 阅读 `rocc/roccAnalog.h`，理解 Vanadis/RoCC/array/globalmemory/scheduler 的连接关系。
3. 阅读 `workercmdproc/workercmdproc.h`，理解 worker-local command progression。
4. 阅读 `requestscheduler/requestscheduler.h`，理解 transfer request 和 scheduler message。
5. 选择一个 small case 作为 JANUS 第一个修改目标。

