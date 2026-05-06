# ch06-10 撤单、批量撤单和失败重试

[返回本章](README.md)

## 本节目标

- 把 撤单、批量撤单和失败重试 转换成具体的 Spot 应用状态、PTB 参数和链上查询。
- 能指出本节功能调用哪些 `pool.move` 入口，以及需要哪些 BalanceManager 权限对象。
- 能为失败交易设计 dry run、错误提示和状态刷新策略。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：订单生命周期事件、执行策略校验和用户可见错误来源。
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)：订单锁定余额、取消退款和 live order 字段。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 正文

`pool.cancel_order` 要求订单属于当前 `balance_manager`，否则 `state.process_cancel` 会失败。取消后 `book.cancel_order` 移除订单，`process_cancel` 计算退款，`vault.settle_balance_manager` 把锁定资产退回 manager，订单发出 canceled 事件。

批量撤单有三类入口：

- `cancel_orders`：任一订单失败则整笔交易失败。
- `cancel_live_order`：如果订单不存在、已成交、已取消或不属于该 manager，则直接跳过。
- `cancel_live_orders`：批量跳过无效订单，适合前端重试和机器人清理。

生产应用对用户手动撤单可用严格 `cancel_order`，对周期性清理应使用 live 版本。

补充阅读：撤单、批量撤单和失败重试 应从应用状态机倒推链上入口。用户在 UI 或 CLI 中看到的是市场、余额、订单、交易提交状态；真正提交时仍然落到 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的固定入口和 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额。

精度和池子发现是所有交易动作的前置条件。用户输入的十进制价格和数量必须转换为 tick/lot 对齐的整数，再交给 `pool.move`，否则错误会在链上校验阶段暴露。

## 开发要点

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 检查问题

- 撤单、批量撤单和失败重试 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
