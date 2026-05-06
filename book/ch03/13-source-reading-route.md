# ch03-13 源码阅读路线

[返回本章](README.md)

## 先抓住结构

读“源码阅读路线”时先画边界。一个真实协议最容易读乱的地方，不是函数太多，而是不知道 Pool、Book、State、Vault 和 BalanceManager 各自负责哪一段。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)
- [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

建议按以下顺序阅读：

1. `pool.move`：先看 public 函数，理解外部调用面。
2. `book/book.move`：看 `Book` 字段、`create_order`、`match_against_book`、`inject_limit_order`。
3. `book/order_info.move`：看订单状态、订单类型约束、fill 事件生成。
4. `book/order.move` 和 `book/fill.move`：看 maker 订单如何变成 Fill。
5. `state/state.move`：看 fill 如何转换为账户余额、费用和历史数据。
6. `vault/vault.move`：看净额如何转成 BalanceManager 存取款。
7. `registry.move` 和 `balance_manager.move`：补齐版本、权限和账户模型。

## 阅读补充

推荐路线是先读 `pool.move` 的 public 函数签名，再读 `book.move` 的撮合，再读 `state.move` 的费用和账户更新，再读 `vault.move` 的资产结算，最后读事件和 Indexer 查询。这条路线对应真实交易顺序，比按文件名 alphabet 读更有效。

每读完一个模块都写一句“它不负责什么”。例如 Book 不负责资产保管，Vault 不负责价格排序，Indexer 不负责交易执行。负面边界能帮助你在调试时快速排除错误归因。

## 工程判断

- 每次跳文件都记录触发跳转的函数名或返回值。
- 阅读笔记要保留“不负责”边界，减少架构误解。
- 查询和 Indexer 放在最后读，因为它们解释结果，不决定交易原子性。

## 读完以后问自己

- 为什么阅读路线从 Pool 开始？
- Book、State、Vault 的阅读顺序对应交易中的哪些阶段？
- 为什么 Indexer 不适合作为第一源码入口？
