# ch03-09 DEEP token 在费用、staking、rebate 中的位置

[返回本章](README.md)

## 本节目标

- 理解 DEEP 如何参与支付费用、stake、rebate 和销毁路径。
- 能沿“DEEP token 在费用、staking、rebate 中的位置”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/vault/deep_price.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/deep_price.move)
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)
- [packages/deepbook/sources/state/history.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/history.move)
- [packages/token](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/token)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

DeepBookV3 中 DEEP 出现在三条路径：

- 下单费用：`pay_with_deep` 为 true 时，`pool::place_order_int` 从 `deep_price` 生成 `OrderDeepPrice`，`order_info.calculate_partial_fill_balances` 计算 DEEP fee。
- 治理和费率：`state::process_create` 根据 stake、交易量和 `trade_params` 计算 taker fee；`state::process_stake`、`process_unstake` 更新账户 stake 和治理投票权。
- rebate 和销毁：`state/history/governance` 记录费用与 rebate，`pool.move` 也包含 `DeepBurned` 事件和 `vault.withdraw_deep_to_burn` 路径。

不要把 DEEP 仅理解成手续费代币。它同时影响用户费率、池治理、maker rebate 和协议统计。

## 阅读补充

DEEP 在 DeepBook 中不是普通展示积分。它会出现在 fee 支付选项、stake 与治理权重、rebate 历史和 burn 路径中；不同路径对应的余额方向和事件也不同。

阅读 DEEP 相关逻辑时先判断它是费用计价、实际支付资产、stake 资产还是统计字段。尤其是 `pay_with_deep` 打开时，订单结算会多一条 DEEP 资金线，需要与 base/quote 分开记录。

## 开发要点

- 展示费用时明确 fee 是 quote 支付还是 DEEP 支付。
- stake/rebate 统计要按 epoch 和账户维度查询 State/History。
- 销毁路径需要追踪 Vault 提币和 burn 事件，不能只看总供应叙述。

## 检查问题

- `pay_with_deep` 会给结算增加哪条资金线？
- stake 如何影响费用或治理相关状态？
- rebate 和 burn 应该分别从哪些模块追踪？
