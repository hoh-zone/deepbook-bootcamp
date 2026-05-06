# ch08-12 TPSL 条件订单

[返回本章](README.md)

## 本节目标

- 理解 TPSL 是存放在 `MarginManager` 内的条件订单队列，而不是 DeepBook 原生订单。
- 能说明创建、触发、取消、失败事件和 permissionless execute 的边界。
- 能把 TPSL 与 reduce-only 平仓、风险率和 oracle 价格保护结合起来设计。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/tpsl.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/tpsl.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`tpsl.move` 保存两组有序条件订单：

- `trigger_below`：价格低于触发价时执行，常用于止损。
- `trigger_above`：价格高于触发价时执行，常用于止盈。

`Condition` 包含 `trigger_below_price` 和 `trigger_price`。`PendingOrder` 支持限价单和市价单字段：`client_order_id`、`order_type`、`self_matching_option`、`price`、`quantity`、`is_bid`、`pay_with_deep`、`expire_timestamp`。

用户通过 `MarginManager.add_conditional_order` 新增 TPSL，通过 `cancel_conditional_order` 或 `cancel_all_conditional_orders` 取消。`execute_conditional_orders` 是 permissionless，任何人都可以触发；执行时会重新计算当前价格，收集可执行、过期、资金不足和价格越界的订单，再发出对应事件。

补充说明：

TPSL 只保存条件和待执行订单参数，真正成交仍要走 `pool_proxy`。因此触发时仍可能因为资金不足、价格越界、Pool 暂停或订单簿流动性不足而失败，失败事件必须进入 UI。

止损单的产品语义通常是降低风险，但如果参数方向错误也可能扩大债务。应用创建 TPSL 前应根据 debt asset、is_bid 和 quantity 判断是否 reduce-only，并提示触发后预估风险率。

## 开发要点

- 后台 worker 可以 permissionless 执行，但不能假设每个触发订单都会成交。
- 同一 manager 的 TPSL 列表要有用户维度唯一 ID，方便取消和事件对账。
- 触发价来自 oracle 检查，不等同于 DeepBook 最终成交价。

## 检查问题

- 这个 TPSL 触发后会买入还是卖出，是否降低当前 debt asset 风险？
- 执行失败事件应该如何反馈给用户和重试 worker？
- `max_orders_to_execute` 如何设置才能控制 gas 和延迟？
