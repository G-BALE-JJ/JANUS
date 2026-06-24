# tests

当前不放测试代码，也不运行端到端验证。

未来测试应按从小到大的顺序添加：

1. descriptor 字段映射测试。
2. P-Golem / D-Golem role 配置测试。
3. KV migration descriptor 自洽测试。
4. SST smoke wrapper 测试。
5. 真实 Golem/SST small case 测试。

当前已经在 `sst/elements/golem/tests/` 下导入精选 Golem 测试参考，用于学习和后续自定义修改。不要再无差别复制完整 upstream `tests`、`artifacts`、`logs` 或构建产物。
