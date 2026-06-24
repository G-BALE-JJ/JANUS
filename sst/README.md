# JANUS-local SST Hardware Tree

## 目标

`sst/` 是 JANUS 项目自己的 SST 硬件工程区。后续 JANUS 自定义的 SST element、P-Golem、D-Golem、KV path、migration fabric、control plane、small tests、configs 和 scripts 都放在这里。

## 非目标

- 不复制 `/data4/jjgong/RISC-V-CIM-Manycore-SST` 源码。
- 不修改 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。
- 当前不编译、不运行 SST。
- 当前不提供真实 C++ component 实现。

## 目录结构

```text
sst/
├── README.md
├── elements/
│   └── golem/
│       ├── README.md
│       ├── IMPORT_NOTES.md
│       ├── golem.cc
│       ├── array/
│       ├── rocc/
│       ├── globalmemory/
│       ├── groupctrl/
│       ├── requestscheduler/
│       ├── workercmdproc/
│       └── tests/
├── tests/
│   ├── README.md
│   └── small/
│       └── README.md
├── configs/
│   └── README.md
└── scripts/
    └── README.md
```

## 与 `docs/sst` 的区别

- `docs/sst/` 记录设计路线、参考架构和迁移计划。
- `sst/` 承载未来真实 JANUS SST 硬件内容。

## 参考基线

主要参考路径：

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem
```

JANUS 后续实现应参考该路径的组件分层，但不要直接在参考仓库中修改。

## 当前硬件目录策略

当前不再维护 `sst/elements/janus` 的过早拆分骨架。JANUS 的第一阶段目标是学习和自定义 Golem SST element，因此本项目内维护一份 local Golem tree：

```text
sst/elements/golem
```

未来的 P-Golem、D-Golem、KV path 和 migration fabric 改动，先在这份 local Golem tree 上逐步演进。
