# JANUS 工程计划

## 目标

建立一个薄的 JANUS 项目骨架，围绕 `/data4/jjgong/RISC-V-CIM-Manycore-SST` 的 Golem/SST 架构整理 P-Golem、D-Golem、KV path、KV migration fabric 和 control plane 的工程边界，为后续真实硬件仿真框架实现做准备。

## 边界

### 当前要做

- 创建项目入口文档和少量架构文档。
- 记录 JANUS 和现有 Golem/SST 组件之间的参考关系。
- 保留 `src/`、`tests/`、`scripts/` 作为未来实现入口，但当前只放 README。
- 建立持续更新的 `task_plan.md`、`findings.md`、`progress.md`。

### 当前不做

- 不运行仿真。
- 不写真实 SST component。
- 不写 Python/C++ 原型实现。
- 不创建 schema/workload/ISA/simulation 泛化目录。
- 不修改 `RISC-V-CIM-Manycore-SST`。
- 不写论文相关内容。

## Phase 0: 项目骨架与参考映射

状态：`complete`

任务：

- [x] 创建 `README.md`，定义 JANUS 当前工程定位。
- [x] 创建 `docs/architecture/` 文档组。
- [x] 创建 `docs/sst/` 文档组。
- [x] 创建 `src/README.md`、`tests/README.md`、`scripts/README.md` 占位入口。
- [x] 记录当前参考事实到 `findings.md`。

## Phase 1: JANUS 架构边界冻结

状态：`pending`

任务：

- [ ] 明确 P-Golem 是否复用现有 `array/` + `rocc/` 抽象，还是需要独立 `janus_pgolem` SST element。
- [ ] 明确 D-Golem 的 KV local memory 是否扩展 `globalmemory/`，还是新增专用 KV memory endpoint。
- [ ] 明确 KV migration fabric 是否先复用 Merlin mesh + DMA path，还是新增专用 fabric component。
- [ ] 明确 host/runtime descriptor 与 worker command descriptor 的最小字段。

## Phase 2: 参考 Golem 路径最小接入设计

状态：`pending`

任务：

- [ ] 选择一个最小参考路径，例如 `tests/small/mvm_noc_int_array` 或 `tests/small/mvm_noc_softmax_cpu`。
- [ ] 记录该路径的 build/run/config/artifact 入口。
- [ ] 梳理哪些环境变量和 runtime descriptor 可以复用。
- [ ] 写出 JANUS 第一个 smoke case 的目标，不执行。

## Phase 3: JANUS-specific runtime descriptor 草案

状态：`pending`

任务：

- [ ] 定义 P-Golem prefill descriptor 草案。
- [ ] 定义 D-Golem decode/KV descriptor 草案。
- [ ] 定义 P->D KV migration descriptor 草案。
- [ ] 对照 `workercmdproc/` 和 `requestscheduler/` 的现有结构检查字段是否可落地。

## Phase 4: 真实 SST component 接入计划

状态：`pending`

任务：

- [ ] 决定 fork existing Golem element 还是在 JANUS 中新建 wrapper。
- [ ] 决定先修改 topology builder 还是先新增 runtime small test。
- [ ] 明确第一个可运行目标的输入、输出和停止条件。
- [ ] 在用户确认后再进入实现。

## 错误与风险记录

| 风险 | 影响 | 当前处理 |
|---|---|---|
| 过早创建泛化 schema/workload/ISA 目录 | 项目偏离真实 SST/Golem 接入 | 已明确不创建 |
| 直接跑 SST | 容易被环境和工具链问题阻塞 | 当前不运行验证 |
| 不区分 P-Golem/D-Golem 与现有 Golem 组件边界 | 后续实现容易重写已有能力 | 通过 `golem-reference-map.md` 固化映射 |

