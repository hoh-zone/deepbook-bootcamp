# ch01-12 第一个读者任务：查询池子对象

[返回本章](README.md)

## 先看问题

先把“第一个读者任务：查询池子对象”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 本节坐标

本节先把源码范围缩到最少。读的时候只抓产品边界、对象名称、函数入口和事件线索；后面进入交易路径时，再回到这些入口核对细节。

- `book/ch01/code/s03-first-query/README.md`
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)

这一组入口用来校准产品边界：先看它们暴露哪些对象和动作，再判断本节概念最终会落到哪条交易或数据路径。

## 建立直觉

选择一个真实 DeepBook 池子对象 ID，设置为 `POOL_ID`，然后使用 Sui CLI 或 TypeScript SDK 查询对象。最小目标不是下单，而是确认对象存在、网络正确、字段能被读取。

CLI 方向：

```bash
sui client object "$POOL_ID" --json
```

TypeScript 方向使用 `@mysten/sui/client` 的 `getObject`，传入 `showType` 和 `showContent`。如果返回 `object not found`，先检查 `NETWORK` 和 `RPC_URL`，再检查池子 ID 是否属于同一网络。

## 阅读补充

第一个任务选择查询 Pool，是因为它不需要签名交易，却能验证 RPC、对象 ID、package 类型和字段解析。查询结果里重点看对象类型是否包含 `Pool<BaseAsset, QuoteAsset>`，字段中是否能看到 `book`、`state`、`vault` 或版本相关结构。

读查询输出时不要只看“有返回”。要把返回类型与 `pool.move` 中的 `Pool` 定义对照，把对象 ID 与 Registry 或官方配置对照，把网络与 RPC URL 对照。三者一致，后续才适合进入下单练习。

## 落地判断

- 查询脚本输出必须包含 RPC、Pool ID 和对象类型。
- 对象存在但类型不匹配时，优先检查是否拿错池或网络。
- 只读查询不能证明余额和权限足够，只能证明对象入口可达。

## 读完以后问自己

- 查询 Pool 对象可以验证哪三类配置？
- 对象类型里的 base/quote 泛型为什么重要？
- 为什么第一步不直接发送下单交易？
