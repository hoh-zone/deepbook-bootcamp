# ch03-07 BalanceManager：交易账户抽象

[返回本章](README.md)

## 本节目标

- 理解交易账户、cap/proof 和多资产余额管理。
- 能沿“BalanceManager：交易账户抽象”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

`balance_manager.move` 把交易余额从钱包 owned coin 中抽象出来。`BalanceManager` 内部用 `Bag` 按 `BalanceKey<T>` 保存多资产余额，并记录 `owner` 和 allow list。

下单需要：

- `&mut BalanceManager`
- `&TradeProof`
- `Pool` 根据订单结果调用 `Vault::settle_balance_manager`
- Vault 内部调用 `balance_manager.validate_proof(trade_proof)` 和 `withdraw_with_proof/deposit_with_proof`

高频交易系统通常使用同一个 BalanceManager 跨多个池交易，这样可以避免每笔订单都拆分钱包 coin。生产系统必须做好 BalanceManager 的权限边界：默认 owner 可生成 proof，TradeCap 持有人也可交易，但 TradeCap 是 owned object，会受到对象并发限制。

## 阅读补充

BalanceManager 不是钱包余额的别名，而是 DeepBook 交易账户。它可以保存多资产余额，并通过 owner、cap 和 proof 支持授权交易；Pool/Vault 只认通过校验的证明和账户余额，不认前端登录态。

开发上应把“充值到 BalanceManager”和“用 BalanceManager 下单”拆成两个流程。前者改变账户内部余额，后者消耗或增加交易账户余额并可能创建 open order。

## 开发要点

- 下单前检查 BalanceManager 余额和 TradeProof，而不是只检查 wallet coin。
- 授权交易要记录 cap 归属和 custom owner 场景。
- 余额展示要区分 available、open order 锁定和已结算可提取。

## 检查问题

- BalanceManager 与钱包地址有什么关系但为什么不等同？
- TradeProof 在 Vault 结算时起什么作用？
- 用户充值成功后为什么仍可能无法下单？
