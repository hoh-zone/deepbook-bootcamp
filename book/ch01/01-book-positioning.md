# ch01-01 本书定位

[返回本章](README.md)

## 本节目标

- 明确这本书服务于 DeepBook 应用开发，而不是只讲交易所概念或 Move 语法。
- 能沿“本书定位”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md)
- [packages/deepbook/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/README.md)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

本书面向已经会写基础程序、准备进入 Sui Move 和 DeepBook 应用开发的读者。你会同时接触五类工程任务：Move 合约阅读、TypeScript SDK 交易构造、后端服务、Indexer 数据表、前端交易面板。

本章先建立全局坐标。源码入口是 [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md) 和 [packages/deepbook/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/README.md)。这两个文件明确 DeepBookV3 是构建在 Sui 上的 CLOB，并说明 `BalanceManager`、`Pool` 和 DEEP 费用的角色。

开发实践上，后续每章都会绑定 GitHub 源码，不把“交易所”“账户”“事件”停留在概念层。读者应把书稿仓库 `deepbook-bootcamp` 当作练习区，把 [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0) 当作权威源码。

## 阅读补充

定位这本书时，先把“读源码”和“构造交易”放在同一条线上：概念章节负责建立对象边界，Move 章节负责读懂资源和能力，架构章节开始把 `Pool`、`Book`、`State`、`Vault`、`BalanceManager` 串成交易路径。这样后续遇到 SDK、Indexer 或前端示例时，不会把链上对象和链下读模型混成一个账户系统。

## 开发要点

- 每章练习都要能回到 DeepBookV3 源码路径，而不是只引用书稿里的摘要。
- 遇到“账户”“池子”“订单”这类词，先判断它在 Move 对象、事件还是链下表里出现。
- 记录本书练习区和权威源码区的路径，避免在 bootcamp 仓库里修改上游协议源码。

## 检查问题

- 这本书的权威协议源码入口是哪两个 README？
- 为什么第一章不直接从撮合算法开始？
- 遇到书稿与源码不一致时，应该以哪个目录为准？
