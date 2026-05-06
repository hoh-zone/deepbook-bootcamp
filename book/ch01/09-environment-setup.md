# ch01-09 环境安装

[返回本章](README.md)

## 先看问题

这一节不急着进源码。先用“环境安装”回答一个更基础的问题：如果要构建真实交易应用，哪些概念必须先讲清楚，哪些细节可以等到后面再拆。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- `book/ch01/code/s01-env-check/README.md`
- [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)
- [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

最小开发环境包括：

- Sui CLI：执行 `sui client`、`sui move build`、`sui move test`。
- Node.js 和 pnpm：运行 TypeScript SDK 示例，优先用 `pnpm tsx`。
- Rust：构建或阅读 [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer) 和 [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)。
- PostgreSQL：运行 Indexer 或本地事件表实验。

本章示例 `book/ch01/code/s01-env-check` 给出环境检查命令。检查通过不代表交易一定能成功；还需要 RPC、gas coin、对象 ID 和网络一致。

## 阅读补充

环境检查的目的不是追求工具越多越好，而是覆盖四类任务：Move 构建测试、TypeScript 交易构造、Rust 服务阅读/运行、PostgreSQL 事件表实验。缺任何一类，后续章节都可以先跳过对应练习，但要在笔记里标注不可执行的原因。

安装后优先运行只读命令，例如 `sui client active-env`、`sui move build`、`node --version`、`rustc --version`，再进入会发交易或写数据库的练习。这样可以把环境问题和协议逻辑问题分开。

## 落地判断

- 把环境检查输出保存到本章练习记录，便于后续复现。
- 先做只读检查，再做链上交易或本地数据库写入。
- Sui CLI 和 Move.toml edition 不匹配时，优先排查工具链版本。

## 读完以后问自己

- 本章环境检查覆盖哪四类工具？
- 为什么通过环境检查不代表交易一定能成功？
- Move 构建失败时应先看 CLI 版本还是前端代码？
