# ch09-10 TPSL 订单

[返回本章](README.md)

## 先看用户路径

这里先从用户动作往回看。“TPSL 订单”在应用里通常是一组 PTB 和状态刷新，不是一条孤立 move call；每一步都要同时考虑余额、债务和风险率。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/tpsl.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/tpsl.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

TPSL 是 manager 内部的条件订单，不是 DeepBook 原生挂单。用户创建 TPSL 时，应用需要构造：

- `conditional_order_id`：用户维度唯一 ID。
- `Condition`：`trigger_below_price` 和 `trigger_price`。
- `PendingOrder`：限价或市价参数，包括 `quantity`、`is_bid`、`pay_with_deep`、过期时间等。

执行是 permissionless。后台 worker 可以定时调用 `execute_conditional_orders`，每次限制 `max_orders_to_execute`，避免单笔交易 gas 不可控。

UI 要展示四类终态事件：

- `ConditionalOrderExecuted`
- `ConditionalOrderCancelled`
- `ConditionalOrderInsufficientFunds`
- `ConditionalOrderPriceOutOfBounds`

创建 TPSL 前必须检查它是否会扩大风险。如果是止损平仓，应优先使用 reduce-only 语义设计交易方向。

## 工程旁白

TPSL worker 只负责在条件满足时提交执行交易，不保证成交成功。它仍然要面对 oracle 价格保护、资金不足、Pool 暂停、订单簿深度不足等失败面，所以事件订阅和重试策略与创建订单同等重要。

止盈止损在 Margin 场景里应优先服务降风险。借 quote 开多的止损通常是卖出 base 换 quote，借 base 开空的止损通常是买回 base；方向反了会扩大债务风险。

## Margin 应用判断

- 创建表单用 debt asset 推导推荐方向，并允许用户明确确认非 reduce-only 订单。
- worker 执行批量订单时限制 `max_orders_to_execute`，并记录每个失败事件。
- 取消 TPSL 后从 UI 列表移除前等待链上事件或交易确认。

## 动手检查

- 这个 TPSL 是止盈、止损还是可能扩大仓位的条件开仓？
- 触发价格、订单价格和 oracle 价格保护之间是什么关系？
- worker 失败后用户看到的是等待重试、资金不足还是价格越界？
