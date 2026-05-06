# ch01-10 检查 DeepBook 源码仓库结构

[返回本章](README.md)

## 先看问题

先把“检查 DeepBook 源码仓库结构”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md)
- [packages](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)
- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

当前源码仓库的顶层重点目录：

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)：DeepBookV3 spot CLOB Move 包。
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)：Margin Move 包。
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)：Predict Move 包。
- [packages/token](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/token)：DEEP token 包。
- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)：运维和交易脚本。
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)：Indexer。
- [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)：查询服务。

本章示例 `book/ch01/code/s02-repo-map` 给出生成模块清单的方法。

## 阅读补充

第一次检查仓库结构时，不要急着打开所有文件。先确认包目录、服务目录和脚本目录的边界，再把 README 中的模块名映射到 `sources` 下的具体 Move 文件。这样后续引用源码时能少走很多搜索弯路。

如果本地仓库与书稿列出的结构不同，先检查分支和版本，再更新个人笔记。不要直接修改 ch01 的源码地图去适配本机临时状态，除非这是明确的文档维护任务。

## 落地判断

- 用目录层级先分产品，再用文件名分模块职责。
- 读到陌生模块时先回 README 或 `Move.toml` 找包级说明。
- 脚本目录只说明运维和调用方式，不等于协议状态定义。

## 读完以后问自己

- `packages/deepbook` 和 `crates/indexer` 的职责差异是什么？
- 为什么脚本目录不能作为协议真相来源？
- 源码结构变动时应先检查什么？
