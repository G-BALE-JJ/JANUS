# Golem Import Notes

## Import purpose

This directory is a JANUS-local learning and customization copy of the Golem SST element.

The current project goal is to understand, modify, and eventually extend the Golem SST element inside the JANUS repository without changing the upstream/reference project.

## Upstream source

Imported from:

```text
/data4/jjgong/RISC-V-CIM-Manycore-SST/src/sst/elements/golem
```

Observed upstream commit:

```text
9852510c3832e7df9fff30fd18d5d53fb22ab269
```

Important: the upstream working tree was not clean at import time. These upstream files were already modified before this import and were copied from the current working tree state:

```text
src/sst/elements/golem/tests/configs/10_core_gemm.env
src/sst/elements/golem/tests/configs/40_debug_io.env
src/sst/elements/golem/tests/small/mvm_noc_int_array/Makefile
```

There was also an upstream untracked file:

```text
summary.md
```

`summary.md` was not imported.

## Imported scope

Core element files:

```text
golem.cc
globalmemory.cc
globalmemory.h
configure.m4
Makefile.am
UPSTREAM_README.md
```

Core subdirectories:

```text
array/
rocc/
globalmemory/
groupctrl/
requestscheduler/
workercmdproc/
```

Selected test/reference files:

```text
tests/architecture/
tests/configs/
tests/small/mvm_noc_int_array/
tests/small/mvm_noc_softmax_cpu/
tests/small/golem_operator_api.h
tests/run_noc_dma_pipeline.sh
tests/golem_dtype.py
```

## Excluded scope

The import intentionally excludes generated or runtime artifacts:

```text
.deps/
.libs/
*.o
*.lo
*.la
Makefile
Makefile.in
__pycache__/
artifacts/
logs/
stats/
hbm dumps
```

## Policy

- Keep upstream copyright headers.
- Do not edit `/data4/jjgong/RISC-V-CIM-Manycore-SST` for JANUS work.
- Make JANUS-specific changes inside this repository under `sst/elements/golem`.
- Prefer small, traceable changes over copying additional upstream build outputs.
- When re-syncing from upstream, update this file with the upstream commit and dirty-state notes.

