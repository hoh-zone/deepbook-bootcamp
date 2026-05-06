# ch06-04 钱包连接和交易签名

[返回本章](README.md)

## 先从用户动作开始

先从用户动作开始。“钱包连接和交易签名”在产品里会表现为按钮、表单、状态刷新或错误提示；链上仍然会落到固定的 Pool 和 BalanceManager 入口。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 从应用反推链上

DeepBook 交易通常是 PTB。钱包只签名交易，资产扣划发生在 Move 调用里。典型 PTB 顺序是：

1. 选择 `BalanceManager` shared/owned object。
2. 创建 `TradeProof`：owner 用 `generate_proof_as_owner`，机器人用 `generate_proof_as_trader`。
3. 调用 `pool.place_limit_order` 或 `pool.place_market_order`。
4. 可选调用 `pool.withdraw_settled_amounts` 把历史 settled 余额落回 manager。

不要把 proof 当作长期凭证保存。它是交易内临时值，适合在 PTB 中构造后立即传入 pool。

> **工程旁白**：Spot 应用先看用户状态机：市场、余额、订单、提交、确认。链上提交仍然落到 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 和 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)；UI 只是把这些固定入口组织成用户能理解的交易动作。

PTB 构造要把 type arguments、pool 对象、manager/proof、价格、数量、过期时间和 self-match 策略放在同一个检查清单中。签名前 dry run 一次，既能估算 Gas，也能把 Move abort 提前翻译成用户能理解的错误。

## 写应用时

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 动手检查

- 钱包连接和交易签名 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
