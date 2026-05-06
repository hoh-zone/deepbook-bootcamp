# ch06-06 限价买单和限价卖单 PTB

[返回本章](README.md)

## 本节目标

- 把 限价买单和限价卖单 PTB 转换成具体的 Spot 应用状态、PTB 参数和链上查询。
- 能指出本节功能调用哪些 `pool.move` 入口，以及需要哪些 BalanceManager 权限对象。
- 能为失败交易设计 dry run、错误提示和状态刷新策略。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：订单生命周期事件、执行策略校验和用户可见错误来源。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 正文

`pool.place_limit_order` 参数含义：

- `client_order_id`：客户端自定义 ID，用于事件和本地订单映射。
- `order_type`：no restriction、IOC、FOK、post-only 等常量。
- `self_matching_option`：自成交处理策略。
- `price`：限价，买单最高愿付，卖单最低愿收。
- `quantity`：base 数量。
- `is_bid`：true 为买单，false 为卖单。
- `pay_with_deep`：是否用 DEEP 支付手续费。
- `expire_timestamp`：订单过期毫秒时间戳。

资金路径：`place_order_int` 创建 `OrderInfo`，`book.create_order` 撮合，`state.process_create` 计算 owed/settled，`vault.settle_balance_manager` 从 manager 扣 quote/base/DEEP 或把成交所得转回 manager，随后发出订单事件。

补充阅读：限价买单和限价卖单 PTB 应从应用状态机倒推链上入口。用户在 UI 或 CLI 中看到的是市场、余额、订单、交易提交状态；真正提交时仍然落到 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的固定入口和 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额。

PTB 构造要把 type arguments、pool 对象、manager/proof、价格、数量、过期时间和 self-match 策略放在同一个检查清单中。签名前 dry run 一次，既能估算 Gas，也能把 Move abort 提前翻译成用户能理解的错误。

## 开发要点

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 检查问题

- 限价买单和限价卖单 PTB 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
