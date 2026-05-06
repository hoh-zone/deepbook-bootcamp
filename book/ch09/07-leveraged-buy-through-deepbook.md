# ch09-07 通过 DeepBook Pool 执行杠杆买入

[返回本章](README.md)

## 本节目标

- 实现借 quote 后通过 DeepBook Pool 买入 base 的 PTB 和交易前模拟。
- 能展示成交均价、滑点、价格保护、借款后风险率和成交后风险率。
- 能确保杠杆买入走 `pool_proxy` 而不是直接调用 DeepBook 原始入口。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)
- `book/ch09/code/s03-borrow-and-trade/README.md`

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

杠杆买入常见路径：

1. 用户有 quote 抵押，如 USDC。
2. 调 `borrow_quote` 借入更多 USDC。
3. 调 `pool_proxy.place_market_order` 或 `place_limit_order` 买入 base。
4. 调 `pool_proxy.withdraw_settled_amounts` 结算成交资产。
5. 重新读取 `manager_state` 和风险率。

`pool_proxy.place_market_order` 会先根据订单簿计算 effective price，再调用 `registry.assert_price`。如果订单簿没有足够流动性，会触发 `ENoLiquidityInOrderbook` 或 DeepBook Pool 侧错误。

交易前模拟要输出三类信息：

- 预估成交均价和滑点。
- 借款后风险率和成交后风险率。
- 价格保护失败、流动性不足、风险率不足时的用户提示。

补充说明：

杠杆买入的债务通常是 quote。base 价格下跌会降低资产价值并压低风险率；quote 利息增长也会逐步抬高 debt amount。因此交易确认页要同时显示价格风险和借款成本。

市价买入最容易在订单簿深度不足时产生失败或过大滑点。`pool_proxy` 会做 effective price 与 oracle 检查，应用侧也应在提交前用订单簿快照给出最坏可接受价格。

## 开发要点

- 买入 PTB 包含 borrow quote、place order、settle/refresh 三个逻辑步骤。
- 所有价格和滑点提示都要标明基于当前订单簿，最终以 dry run 为准。
- 成交后如果没有立即还款，继续展示 quote debt 的利息累计。

## 检查问题

- 买入后 debt asset 是否仍是 quote，base 下跌对风险率有什么影响？
- 订单失败是 risk ratio 不足、oracle price out of bounds 还是 orderbook liquidity 不足？
- 交易完成后是否刷新 settled amounts 和 open orders？
