# ch06-02 交易对发现、池子选择和显示

[返回本章](README.md)

## 先从用户动作开始

先从用户动作开始。“交易对发现、池子选择和显示”在产品里会表现为按钮、表单、状态刷新或错误提示；链上仍然会落到固定的 Pool 和 BalanceManager 入口。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：`iter_orders` 分页读取 bids/asks，服务订单簿和深度展示。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 从应用反推链上

池子由泛型类型 `Pool<BaseAsset, QuoteAsset>` 表示。`PoolCreated` 事件记录 `pool_id`、`tick_size`、`lot_size`、`min_size`、`taker_fee`、`maker_fee`、`whitelisted_pool`。如果使用 registry，需要通过 `registry.get_pool_id<BaseAsset, QuoteAsset>` 获取已注册池子 ID。

前端显示时，base 是下单数量单位，quote 是价格单位。`place_limit_order` 的 `quantity` 始终以 base asset 计；买单支付 quote，卖单支付 base。价格、数量都使用链上整数精度，不能直接用浮点数提交。

开发注意事项：

- 池子类型必须与用户选择的 base/quote 完全一致，反向交易对不是同一个泛型类型。
- stable pool、whitelisted pool 会影响默认费率。
- 提交前检查 `registered_pool()` 和版本状态，避免 stale pool。

> **工程旁白**：Spot 应用先看用户状态机：市场、余额、订单、提交、确认。链上提交仍然落到 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 和 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)；UI 只是把这些固定入口组织成用户能理解的交易动作。

精度和池子发现是所有交易动作的前置条件。用户输入的十进制价格和数量必须转换为 tick/lot 对齐的整数，再交给 `pool.move`，否则错误会在链上校验阶段暴露。

## 写应用时

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 动手检查

- 交易对发现、池子选择和显示 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
