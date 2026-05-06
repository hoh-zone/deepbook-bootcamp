# ch00-07 DEEP、费用、staking 与治理

[返回本章](README.md)

DeepBookV3 不只是一个撮合引擎，它还围绕 DEEP token 建立费用、staking、maker rebate 和池级治理机制。对应用开发者来说，这些机制直接影响交易前余额检查、费用展示、做市收益、治理页面和后台索引。

费用层面，DeepBook 的交易会涉及 maker fee、taker fee、DEEP 支付和 rebate。仓库 README 明确强调 DEEP 在费用和治理中的作用。源码中，费用参数、stake required、rebate 和历史成交量主要分布在 [state](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state)、[trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)、[history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move) 和 [governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)。

Staking 的作用有两类。第一，用户在某个池中 stake DEEP 后，可能获得费用和 maker rebate 相关权益。第二，stake 是参与治理的基础。池级治理不是随意改任意参数，而是围绕 taker fee、maker fee 和 stake required 等交易参数进行。

`Pool` 中的公开函数包括 `stake`、`unstake`、`submit_proposal`、`vote` 和 `claim_rebates`。这说明 DeepBook 的治理和收益领取不是一个外部后台动作，而是合约公开接口的一部分。Indexer 也需要记录这些事件，否则前端很难展示用户历史、当前投票、rebate 领取和参数变化。

开发时最容易漏掉的是费用货币和交易前检查。一个用户有 base 和 quote 并不代表能成功下单，因为订单可能还需要 DEEP 支付费用。另一个常见错误是把 fee tier 写死在前端。更稳妥的做法是从链上或 Server 查询池参数、用户 stake 状态和费用相关数据，在交易前用 dry run 验证最终路径。

## 本节检查

- [ ] 能说明 DEEP 在交易费用中的作用。
- [ ] 能解释 staking 与治理的关系。
- [ ] 能列出 `Pool` 中治理相关的公开函数。
