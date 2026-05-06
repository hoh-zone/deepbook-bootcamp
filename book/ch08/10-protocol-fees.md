# ch08-10 protocol_fees

[返回本章](README.md)

## 先看风险边界

这里先把问题放到风险控制面里。“protocol_fees”不是在 DeepBook 旁边加一层 UI，而是把 registry、manager、oracle、borrow/supply state 接进同一条链上风控路径。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/protocol_fees.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_fees.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

`protocol_fees.move` 把利息中的协议 spread 拆成三类账：

- maintainer fees：维护者可通过 `withdraw_maintainer_fees` 提取。
- protocol fees：协议管理员可通过 `withdraw_protocol_fees` 提取。
- referral fees：供应 referral 按 shares 追踪。

`MarginPool.supply` 和 `withdraw` 会根据用户 referral 调整 `ProtocolFees` 内部 shares。`margin_state.update` 返回本次累计的 protocol fees，调用方再把它记入费用模块。阅读这部分时要区分两种 shares：借贷池的 supply/borrow shares 表示本金权益，费用模块的 shares 表示 referral 分配权重。

## 工程旁白

费用模块把“利息增长”和“费用领取”解耦。借款人看到 debt 增长，供应者看到 shares 对应 amount 增长，协议和维护者则通过 fee 状态累积可领取金额。

清算奖励不应计入普通借款 APR。清算时的 user liquidation reward 与 pool liquidation reward 来自 `PoolConfig`，它们影响清算 payout 和坏账覆盖，不是每秒 accruing 的利息费用。

## 风控判断

- 收益页把 borrow APR、supply APR、protocol fee 和 liquidation reward 分栏展示。
- 统计 referral 收益时读取供应 position 中的 referral，而不是交易 manager。
- 清算事件单独入账，避免把清算奖励误算为常规利息。

## 动手检查

- 本次费用来自持续借款利息，还是来自清算流程？
- protocol spread 变化会影响供应者 APR、借款人 APR 还是两者展示？
- referral 费用应按 supply position 还是 manager owner 归属？
