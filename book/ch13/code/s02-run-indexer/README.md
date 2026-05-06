# s02 run indexer

在 DeepBook 源码目录运行：

```bash
cd deepbookv3
DATABASE_URL="postgresql://postgres:postgrespw@localhost:5432/deepbook" \
cargo run --package deepbook-indexer -- --env testnet --packages deepbook deepbook-margin
```

只索引核心 DeepBook：

```bash
cargo run --package deepbook-indexer -- --env testnet --packages deepbook
```

Margin 事件在网络部署状态上可能与核心 DeepBook 不一致，启动前先确认目标网络是否已部署对应 package。

