# tests

当前不放测试代码，也不运行端到端验证。

未来测试应按从小到大的顺序添加：

1. descriptor 字段映射测试。
2. P-Golem / D-Golem role 配置测试。
3. KV migration descriptor 自洽测试。
4. SST smoke wrapper 测试。
5. 真实 Golem/SST small case 测试。

当前不要复制 `RISC-V-CIM-Manycore-SST/src/sst/elements/golem/tests`。需要时只引用其路径或新增明确的 JANUS small case。

