# ch06-13 最小命令行交易终端

[返回本章](README.md)

## 先从用户动作开始

这一节把“最小命令行交易终端”当作应用流程来读：先看用户要完成什么，再反推 PTB 需要哪些对象、参数、dry run 和失败处理。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 从应用反推链上

CLI 交易终端应提供 `markets`、`balances`、`book`、`limit`、`market`、`cancel`、`fills` 命令。所有命令都接受 `--pool` 和 base/quote type 参数，避免把不同泛型池混在一起。

命令行输出应使用整数和人类可读两种格式。提交交易时打印 digest、order ID、client order ID 和事件摘要，方便和 indexer 对账。

> **工程旁白**：Spot 应用先看用户状态机：市场、余额、订单、提交、确认。链上提交仍然落到 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 和 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)；UI 只是把这些固定入口组织成用户能理解的交易动作。

自动化交易循环应把“取行情 -> 计算目标报价 -> dry run -> 提交 -> 监听事件 -> 刷新库存”拆成独立步骤。任何一步出现余额不足、订单已消失或版本暂停，都应该停止当前轮报价而不是继续重试。

## 写应用时

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 动手检查

- 最小命令行交易终端 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
