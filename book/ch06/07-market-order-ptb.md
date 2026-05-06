# ch06-07 市价买入和市价卖出 PTB

[返回本章](README.md)

## 先从用户动作开始

这一节把“市价买入和市价卖出 PTB”当作应用流程来读：先看用户要完成什么，再反推 PTB 需要哪些对象、参数、dry run 和失败处理。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)：订单生命周期事件、执行策略校验和用户可见错误来源。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 从应用反推链上

`pool.place_market_order` 也是以 base quantity 下单。买入时 `is_bid = true`，价格自动是 `constants::max_price()`；卖出时 `is_bid = false`，价格自动是 `constants::min_price()`。函数内部把 order type 设为 `immediate_or_cancel`，未成交数量会取消。

市价单前必须调用 dry run：

- `get_quote_quantity_in(target_base_quantity, pay_with_deep, clock)` 估算买入目标 base 需要多少 quote。
- `get_base_quantity_in(target_quote_quantity, pay_with_deep, clock)` 估算卖出得到目标 quote 需要多少 base。
- `get_quantity_out` 或 `get_quantity_out_input_fee` 估算给定输入能拿到多少输出。

设置 `min_quote_out` 或前端滑点保护时，不要只看最优价，要按订单簿深度逐档计算。

> **工程旁白**：Spot 应用先看用户状态机：市场、余额、订单、提交、确认。链上提交仍然落到 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 和 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)；UI 只是把这些固定入口组织成用户能理解的交易动作。

PTB 构造要把 type arguments、pool 对象、manager/proof、价格、数量、过期时间和 self-match 策略放在同一个检查清单中。签名前 dry run 一次，既能估算 Gas，也能把 Move abort 提前翻译成用户能理解的错误。

## 写应用时

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 动手检查

- 市价买入和市价卖出 PTB 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
