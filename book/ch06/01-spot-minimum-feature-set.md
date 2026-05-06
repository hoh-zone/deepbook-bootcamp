# ch06-01 Spot 应用的最小功能集

[返回本章](README.md)

## 先从用户动作开始

这一节把“Spot 应用的最小功能集”当作应用流程来读：先看用户要完成什么，再反推 PTB 需要哪些对象、参数、dry run 和失败处理。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 从应用反推链上

最小 Spot 应用需要六个能力：池子发现、余额管理、下单、撤单、订单簿展示、成交和订单状态刷新。DeepBook 的交易入口都在 `pool.move`：`place_limit_order`、`place_market_order`、`cancel_order`、`cancel_orders`、`cancel_live_order`、`cancel_live_orders`、`cancel_all_orders`。

应用状态应按三类缓存组织：

- 市场状态：pool ID、base/quote type、tick size、lot size、min size、当前 fee 参数。
- 用户状态：wallet coins、`BalanceManager` ID、manager 内 base/quote/DEEP 余额、open orders。
- 交易状态：最近 dry run、待签名 PTB、提交 digest、事件确认结果。

交易按钮不可只依赖 UI 表单校验。提交前必须 dry run PTB，并把 Move abort 翻译成用户能处理的提示。

> **工程旁白**：Spot 应用先看用户状态机：市场、余额、订单、提交、确认。链上提交仍然落到 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 和 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)；UI 只是把这些固定入口组织成用户能理解的交易动作。

精度和池子发现是所有交易动作的前置条件。用户输入的十进制价格和数量必须转换为 tick/lot 对齐的整数，再交给 `pool.move`，否则错误会在链上校验阶段暴露。

## 写应用时

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 动手检查

- Spot 应用的最小功能集 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
