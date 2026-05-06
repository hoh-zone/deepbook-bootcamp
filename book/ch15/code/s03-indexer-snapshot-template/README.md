# s03 indexer snapshot template

Indexer 快照测试流程：

1. 准备 checkpoint fixture。
2. 选择 handler。
3. 执行 `data_test`。
4. 比较 snapshot。
5. 如果事件结构变化，更新 model、migration 和 snapshot。

