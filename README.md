# JANUS

JANUS 是面向 LLM 推理的异构 RISC-V Golem 阵列硬件仿真项目。当前阶段不是论文写作目录，也不是完整仿真器实现目录；它的目标是先把 JANUS 的 P-Golem / D-Golem 架构边界整理清楚，并对齐现有 `/data4/jjgong/RISC-V-CIM-Manycore-SST` 中 Golem/SST 的真实组件组织。

## 当前目标

- 梳理 JANUS 的 P-Golem、D-Golem、KV cache path、KV migration fabric 和 control plane。
- 明确 JANUS 概念如何映射到 `RISC-V-CIM-Manycore-SST/src/sst/elements/golem` 的现有组件边界。
- 为后续真实 SST component、runtime descriptor、NoC/DMA path 和 smoke test 接入准备技术文档。
- 保持项目骨架足够薄，避免在没有真实 SST 框架前引入过多抽象目录。

## 当前非目标

- 不运行 SST 仿真。
- 不实现端到端验证。
- 不修改 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。
- 不创建 `docs/simulation`、`docs/isa`、`docs/workloads`、`docs/development` 或 `schemas`。
- 不做论文相关章节、投稿策略或实验结论。
- 不承诺具体性能数字或加速比。

## 目录结构

```text
JANUS/
├── README.md
├── idea.md
├── task_plan.md
├── findings.md
├── progress.md
├── docs/
│   ├── architecture/
│   │   ├── overview.md
│   │   ├── golem-reference-map.md
│   │   ├── p-golem.md
│   │   ├── d-golem.md
│   │   ├── kv-cache-path.md
│   │   ├── kv-migration-fabric.md
│   │   └── control-plane.md
│   └── sst/
│       ├── reference-architecture-notes.md
│       ├── component-boundary-plan.md
│       └── porting-roadmap.md
├── sst/
│   ├── README.md
│   ├── elements/
│   │   └── golem/
│   │       ├── README.md
│   │       ├── IMPORT_NOTES.md
│   │       ├── Makefile.am
│   │       ├── configure.m4
│   │       ├── golem.cc
│   │       ├── globalmemory.cc
│   │       ├── globalmemory.h
│   │       ├── array/
│   │       ├── rocc/
│   │       ├── globalmemory/
│   │       ├── groupctrl/
│   │       ├── requestscheduler/
│   │       ├── workercmdproc/
│   │       └── tests/
│   │           ├── architecture/
│   │           ├── configs/
│   │           └── small/
│   ├── tests/
│   │   ├── README.md
│   │   └── small/
│   │       └── README.md
│   ├── configs/
│   │   └── README.md
│   └── scripts/
│       └── README.md
├── src/
│   └── README.md
├── tests/
│   └── README.md
└── scripts/
    └── README.md
```

## 参考基线

JANUS 当前以本地 Golem/SST 工程为主要工程参考：

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem
```

重点参考对象包括：

- `array/`：MVM/CIM array 抽象。
- `rocc/`：RoCC coprocessor 接口。
- `globalmemory/`：Golem local/global memory 抽象。
- `groupctrl/`：group-local lightweight control endpoint。
- `requestscheduler/`：transfer request scheduler endpoint。
- `workercmdproc/`：worker-side command processor prototype。
- `tests/architecture/`：SST topology builder、CPU builder、NoC builder。
- `tests/small/`：小型 runtime/test case 组织方式。

## JANUS-local SST 硬件区

`sst/` 是 JANUS 自己的 SST 硬件工程区。它当前用于承载一份 JANUS-local Golem SST element 源码树，作为学习、裁剪和自定义修改 Golem 组件的工作区。

该目录不覆盖、不修改 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。参考仓库只作为 upstream/reference；JANUS 的新增硬件内容在本仓库内独立演进。

`docs/sst/` 与 `sst/` 的职责不同：

- `docs/sst/`：设计文档、参考说明、迁移路线。
- `sst/`：JANUS-local Golem 源码、测试参考、配置参考和未来自定义脚本。

## 建议阅读顺序

1. [idea.md](idea.md)：原始 JANUS 概念。
2. [docs/architecture/overview.md](docs/architecture/overview.md)：JANUS 工程架构视图。
3. [docs/architecture/golem-reference-map.md](docs/architecture/golem-reference-map.md)：JANUS 与现有 Golem/SST 组件边界的映射。
4. [docs/sst/component-boundary-plan.md](docs/sst/component-boundary-plan.md)：后续接入真实 SST component 的边界计划。
5. [sst/README.md](sst/README.md)：JANUS-local SST 硬件工程区说明。
6. [sst/elements/golem/IMPORT_NOTES.md](sst/elements/golem/IMPORT_NOTES.md)：本地 Golem 源码导入范围和来源记录。
7. [task_plan.md](task_plan.md)：当前工程推进计划。
