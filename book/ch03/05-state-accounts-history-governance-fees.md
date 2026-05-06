# ch03-05 State：账户、历史、治理、费用、参数

[返回本章](README.md)

## 先抓住结构

读“State：账户、历史、治理、费用、参数”时先画边界。一个真实协议最容易读乱的地方，不是函数太多，而是不知道 Pool、Book、State、Vault 和 BalanceManager 各自负责哪一段。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)
- [packages/deepbook/sources/state/history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move)
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

`state.move` 的 `State` 包含 `accounts: Table<ID, Account>`、`history: History`、`governance: Governance`。`State::process_create` 是撮合之后的账务中枢：

- 更新治理和历史：`governance.update(ctx)`、`history.update(...)`。
- 处理每个 maker fill：`process_fills` 会更新 maker 账户、成交量和应结算余额。
- 更新 taker 账户：确保账户存在，读取交易量和 stake。
- 根据 `trade_params`、账户 stake、历史交易量和 EWMA 惩罚计算 taker fee。
- 如果新订单剩余数量被插入订单簿，则加入 account open orders。
- 调用 `order_info.calculate_partial_fill_balances` 得到本次 taker 的 `settled` 和 `owed`。

这说明订单状态和资金状态是分阶段写入的：Book 先改变撮合状态，State 再把成交解释为账户余额变化，Vault 最后才移动真实资产。

## 阅读补充

State 是撮合之后的会计和参数层。Book 告诉协议成交了什么，State 决定这些成交如何影响 maker/taker 账户、交易量、stake、费率、rebate 和历史统计。

阅读 State 时建议按函数职责拆表：账户 open orders、已结算余额、历史 epoch 数据、治理参数、费率计算。把这些混成一个“状态更新”会让费用来源和最终 owed/settled 难以解释。

## 工程判断

- 费用问题优先查 trade params、stake、历史交易量和 governance 配置。
- 账户 open orders 与 Vault 资产余额不是同一类状态。
- State 输出的 owed/settled 需要继续交给 Vault 结算。

## 读完以后问自己

- State 为什么不负责订单簿排序？
- maker/taker fee 计算依赖哪些状态？
- `owed` 和 `settled` 为什么还不是最终钱包余额？
