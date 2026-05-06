# s03 indexer snapshot template

Indexer 快照测试流程：

1. 准备 checkpoint fixture。
2. 选择 handler。
3. 执行 `data_test`。
4. 比较 snapshot。
5. 如果事件结构变化，更新 model、migration 和 snapshot。

## 建议命令

```bash
cargo test --package deepbook-indexer snapshot
```

## 快照应覆盖

- 原始 event 字段。
- handler 输出模型。
- PostgreSQL 主键和唯一约束。
- 重放同一 checkpoint 的幂等性。

如果 snapshot 变化，PR 描述必须说明是链上事件变了、handler 映射变了，还是 schema 迁移变了。
