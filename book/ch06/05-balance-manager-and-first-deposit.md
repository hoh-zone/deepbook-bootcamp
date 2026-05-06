# ch06-05 创建 BalanceManager 和首次充值

[返回本章](README.md)

## 本节目标

- 把 创建 BalanceManager 和首次充值 转换成具体的 Spot 应用状态、PTB 参数和链上查询。
- 能指出本节功能调用哪些 `pool.move` 入口，以及需要哪些 BalanceManager 权限对象。
- 能为失败交易设计 dry run、错误提示和状态刷新策略。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 正文

首次交易流程：

1. `balance_manager::new` 创建 manager。
2. 可选 `register_balance_manager`，方便账户查询。
3. `deposit<BaseAsset>`、`deposit<QuoteAsset>`、`deposit<DEEP>` 充值。
4. 生成 proof 并下单。

买单需要 quote 余额和手续费余额；卖单需要 base 余额和手续费余额。当前源码注释说明 `place_limit_order` 当前版本要求 `pay_with_deep` 为 true，应用应优先走 DEEP 手续费路径，并在 dry run 中确认 DEEP 足够。

补充阅读：创建 BalanceManager 和首次充值 应从应用状态机倒推链上入口。用户在 UI 或 CLI 中看到的是市场、余额、订单、交易提交状态；真正提交时仍然落到 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的固定入口和 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额。

精度和池子发现是所有交易动作的前置条件。用户输入的十进制价格和数量必须转换为 tick/lot 对齐的整数，再交给 `pool.move`，否则错误会在链上校验阶段暴露。

## 开发要点

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 检查问题

- 创建 BalanceManager 和首次充值 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
