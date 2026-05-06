# s02-repo-map 源码地图

本示例用于从本地 DeepBook 源码生成模块清单。权威源码目录是 [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0)。

## 运行命令

```bash
cd deepbookv3
find packages/deepbook/sources -type f -name '*.move' | sort
find packages -maxdepth 2 -name Move.toml | sort
find crates -maxdepth 2 -type d | sort
```

## 重点输出

DeepBookV3 spot 包：

```text
packages/deepbook/sources/pool.move
packages/deepbook/sources/balance_manager.move
packages/deepbook/sources/registry.move
packages/deepbook/sources/book/book.move
packages/deepbook/sources/book/order.move
packages/deepbook/sources/book/order_info.move
packages/deepbook/sources/vault/vault.move
packages/deepbook/sources/state/state.move
```

## 阅读顺序

第一遍只看入口：

1. [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md)
2. [packages/deepbook/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/README.md)
3. [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)
4. [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

第二遍再看 `balance_manager.move`、`book/*`、`state/*`、`vault/*`。

## 扩展任务

把 `find` 输出转换成 Markdown 表格，增加“模块职责”和“后续章节”两列，作为全书源码索引的初稿。
