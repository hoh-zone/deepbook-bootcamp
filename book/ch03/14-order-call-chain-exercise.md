# ch03-14 下单调用链练习

[返回本章](README.md)

## 先抓住结构

先把“下单调用链练习”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

用一个限价买单推演源码路径：

- 输入：`is_bid = true`、`price = 1000`、`quantity = 5000`、`order_type = no_restriction`。
- Pool 创建 `OrderInfo`，记录 taker、BalanceManager、订单类型、价格、数量、DEEP fee 选项。
- Book 生成 bid order id，并从 asks 最低价开始扫描。
- 如果 ask price <= bid price，则 `order_info.match_maker` 调用 maker `Order::generate_fill`。
- `Fill` 记录 maker order id、成交价、base/quote 数量、maker/taker fee 标记。
- 完全成交的 maker 从 asks 移除，过期 maker 也移除。
- 如果 taker 剩余数量未完成并且不是 IOC/FOK/Post-only abort 场景，则插入 bids。
- State 更新 maker/taker 账户，计算费用和 owed/settled。
- Vault 在 Pool 与 BalanceManager 之间做净额结算。
- 最后 emit `OrderInfo`、`OrderFilled`、`OrderPlaced` 或 `OrderFullyFilled` 等事件。

## 阅读补充

这个练习建议实际写成表格：每一行是一个阶段，列出当前模块、关键函数、输入、输出、可能 abort 和事件。这样一笔订单能同时训练源码跳转、交易参数理解和失败排查。

复盘时不要只写成功路径。把 IOC/FOK/Post-only、过期 maker、部分成交、余额不足、tick/lot 不合法分别放进旁注，后续做 SDK 或前端时就能把错误提示设计得更准确。

## 工程判断

- 练习输出必须包含控制流、资金流和事件流三列。
- 每个阶段都记录可能 abort 的原因，尤其是参数和权限检查。
- 部分成交场景要说明剩余数量是插入订单簿还是被订单类型取消。

## 读完以后问自己

- 限价买单从 Pool 到 Vault 至少经过哪些模块？
- 部分成交后 `OrderInfo` 为什么还要继续参与 State 处理？
- 哪些订单类型会改变“剩余数量是否挂单”的结果？
