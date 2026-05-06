# s01-env-check 环境检查

本示例用于确认本书需要的基础工具是否可用。它对应正文 `book/ch01/README.md` 的环境安装小节。

## 检查命令

```bash
sui --version
node --version
pnpm --version
rustc --version
cargo --version
psql --version
```

## 期望结果

每条命令都应该输出版本号。`sui` 用于 Move 包构建、测试和链上对象查询；`node` 与 `pnpm` 用于 TypeScript SDK 示例；`rustc` 与 `cargo` 用于阅读和运行 [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)、[crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)；`psql` 用于 Indexer 数据库检查。

## 失败处理

- `sui: command not found`：先安装 Sui CLI，再确认二进制在 `PATH` 中。
- `pnpm: command not found`：用 Corepack 或 npm 安装 pnpm。
- `psql: command not found`：只影响 Indexer 章节，ch01/ch02 的 Move 和对象查询仍可继续。

## 扩展任务

把这些命令整理成 `check.sh`，输出 JSON，例如：

```json
{
  "sui": "sui 1.x",
  "node": "v22.x",
  "pnpm": "10.x",
  "rustc": "rustc 1.x",
  "postgres": "psql 17.x"
}
```
