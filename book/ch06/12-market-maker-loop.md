# ch06-12 做市机器人数据和交易循环

[返回本章](README.md)

## 本节目标

- 把 做市机器人数据和交易循环 转换成具体的 Spot 应用状态、PTB 参数和链上查询。
- 能指出本节功能调用哪些 `pool.move` 入口，以及需要哪些 BalanceManager 权限对象。
- 能为失败交易设计 dry run、错误提示和状态刷新策略。

## 源码关联

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`place_limit_order`、`place_market_order`、`cancel_order(s)`、`cancel_live_order(s)` 等 Spot PTB 入口。
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：创建或复用 `BalanceManager`、deposit/withdraw、`TradeProof` 授权。
- `book/ch06/code/s01-pool-list-cli` 至 `book/ch06/code/s05-mini-terminal`：本章 CLI/PTB 示例的应用侧落点。

## 正文

做市机器人最小循环：

1. 拉取池子参数和订单簿。
2. 拉取自己的 open orders 和 manager 余额。
3. 根据目标价差生成报价。
4. 撤掉偏离报价。
5. 提交新的限价单。
6. 处理 `OrderFilled` 和 `OrderFullyFilled`，更新库存。

机器人应持有 `TradeCap`，不持有 `WithdrawCap`。每轮先 dry run，失败时按 abort code 分类：余额不足暂停报价，版本错误刷新配置，订单已消失则忽略撤单失败。

补充阅读：做市机器人数据和交易循环 应从应用状态机倒推链上入口。用户在 UI 或 CLI 中看到的是市场、余额、订单、交易提交状态；真正提交时仍然落到 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的固定入口和 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额。

自动化交易循环应把“取行情 -> 计算目标报价 -> dry run -> 提交 -> 监听事件 -> 刷新库存”拆成独立步骤。任何一步出现余额不足、订单已消失或版本暂停，都应该停止当前轮报价而不是继续重试。

## 开发要点

- 所有用户输入先做 decimal、tick size、lot size 和 min size 转换，再构造 PTB。
- 提交前 dry run，提交后用 digest、事件和 `order_query.move` 刷新状态。
- UI/CLI 中分开展示 wallet coins、manager 余额、open orders 和最近一次交易状态。

## 检查问题

- 做市机器人数据和交易循环 需要哪些对象 ID、type arguments、cap/proof 和数值参数？
- dry run 失败时应给用户显示哪类提示：余额、精度、订单状态、版本暂停还是 Gas？
- 交易成功后应用应该依赖事件、order query 还是本地缓存更新页面？
