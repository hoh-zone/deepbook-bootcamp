# ch01-08 网络使用方式

[返回本章](README.md)

## 先看问题

先把“网络使用方式”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- `book/ch01/code/s03-first-query/README.md`

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

本书默认用GitHub 源码解释机制，用 testnet 或 mainnet 查询真实对象，用 localnet 做可控实验。主网适合读对象和事件，不适合随意发交易。测试网适合发交易练习，但对象 ID 和包版本会变化。本地网适合 Move 包发布、单元测试和失败案例复现。

脚本配置中可见网络常量，例如 [scripts/config/constants.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/config/constants.ts) 保存了 mainnet/testnet 的 admin cap、upgrade cap 等对象 ID。这些 ID 是脚本运维材料，不等同于所有池子清单。

开发示例中优先通过环境变量传入对象 ID，例如 `POOL_ID`、`RPC_URL`、`NETWORK`，避免把可能变化的链上地址写死到书稿里。

## 阅读补充

DeepBook 对象 ID 与网络强绑定。mainnet 的 Pool、Registry、BalanceManager 不能拿到 testnet RPC 上查询；即使对象 ID 格式合法，RPC 返回的“不存在”也可能只是网络选错，而不是对象被删除。

示例脚本建议总是从 `RPC_URL`、`NETWORK`、`POOL_ID` 读取环境变量，并在输出里打印网络和对象 ID。后续涉及 package version 的章节，还要把 Registry/Pool 的版本开关纳入检查。

## 落地判断

- 每个命令示例都显式列出依赖的网络和对象 ID。
- 查询失败时先核对 RPC 网络，再排查对象权限或字段解析。
- 不要把文档中的对象 ID 当作永久常量，升级和部署会改变入口地址。

## 读完以后问自己

- mainnet Pool ID 用 testnet RPC 查询会出现什么现象？
- 示例环境至少应该暴露哪些变量？
- 为什么版本开关也属于网络使用检查的一部分？
