# JANUS 进展记录

## 2026-06-24

### 已完成

- 明确 JANUS 当前不是论文工作目录。
- 明确当前不跑端到端验证。
- 明确当前不创建 `docs/simulation`、`docs/isa`、`docs/workloads`、`docs/development`、`schemas`。
- 查看 `/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem` 的真实目录结构。
- 识别可参考组件：`array/`、`rocc/`、`globalmemory/`、`groupctrl/`、`requestscheduler/`、`workercmdproc/`、`tests/architecture/`、`tests/small/`。
- 创建 JANUS 精简工程骨架和技术文档。

### 创建的文件

- `README.md`
- `task_plan.md`
- `findings.md`
- `progress.md`
- `docs/architecture/overview.md`
- `docs/architecture/golem-reference-map.md`
- `docs/architecture/p-golem.md`
- `docs/architecture/d-golem.md`
- `docs/architecture/kv-cache-path.md`
- `docs/architecture/kv-migration-fabric.md`
- `docs/architecture/control-plane.md`
- `docs/sst/reference-architecture-notes.md`
- `docs/sst/component-boundary-plan.md`
- `docs/sst/porting-roadmap.md`
- `src/README.md`
- `tests/README.md`
- `scripts/README.md`

### 补充更新

- 用户确认需要在 JANUS 项目内新建自己的 SST 硬件目录，未来自定义 SST 组件在 JANUS 内完成，不影响 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。
- 新增 `sst/` JANUS-local SST 硬件工程区。
- 更新 `README.md`、`task_plan.md`、`findings.md`、`docs/sst/component-boundary-plan.md`，记录该决策。
- 用户进一步确认当前本质只需要 Golem，不需要 `sst/elements/janus` 的过早多分支骨架。
- 将本地硬件源码区收敛为 `sst/elements/golem/`。
- 从参考仓库导入 Golem 核心源码、核心子目录、精选 architecture/config/small test 参考文件。
- 新增 `sst/elements/golem/IMPORT_NOTES.md`，记录 upstream commit、dirty 状态和导入范围。
- 用户确认顶层 `src/` 已不需要；删除 `src/README.md`，硬件源码统一放在 `sst/`。

### 未执行

- 未运行 SST。
- 未运行测试。
- 未修改 `/data4/jjgong/RISC-V-CIM-Manycore-SST`。
- 未修改 `idea.md`。
