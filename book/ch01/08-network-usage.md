# ch01-08 网络使用方式

[返回本章](README.md)

## 本节目标

- 保证 RPC、对象 ID、package version 和示例环境一致。
- 能沿“网络使用方式”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- `book/ch01/code/s03-first-query/README.md`

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

本书默认用GitHub 源码解释机制，用 testnet 或 mainnet 查询真实对象，用 localnet 做可控实验。主网适合读对象和事件，不适合随意发交易。测试网适合发交易练习，但对象 ID 和包版本会变化。本地网适合 Move 包发布、单元测试和失败案例复现。

脚本配置中可见网络常量，例如 [scripts/config/constants.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/config/constants.ts) 保存了 mainnet/testnet 的 admin cap、upgrade cap 等对象 ID。这些 ID 是脚本运维材料，不等同于所有池子清单。

开发示例中优先通过环境变量传入对象 ID，例如 `POOL_ID`、`RPC_URL`、`NETWORK`，避免把可能变化的链上地址写死到书稿里。

## 阅读补充

DeepBook 对象 ID 与网络强绑定。mainnet 的 Pool、Registry、BalanceManager 不能拿到 testnet RPC 上查询；即使对象 ID 格式合法，RPC 返回的“不存在”也可能只是网络选错，而不是对象被删除。

示例脚本建议总是从 `RPC_URL`、`NETWORK`、`POOL_ID` 读取环境变量，并在输出里打印网络和对象 ID。后续涉及 package version 的章节，还要把 Registry/Pool 的版本开关纳入检查。

## 开发要点

- 每个命令示例都显式列出依赖的网络和对象 ID。
- 查询失败时先核对 RPC 网络，再排查对象权限或字段解析。
- 不要把文档中的对象 ID 当作永久常量，升级和部署会改变入口地址。

## 检查问题

- mainnet Pool ID 用 testnet RPC 查询会出现什么现象？
- 示例环境至少应该暴露哪些变量？
- 为什么版本开关也属于网络使用检查的一部分？
