# ch01-06 核心用户角色

[返回本章](README.md)

## 本节目标

- 区分 trader、maker、管理员、应用和链下服务的职责。
- 能沿“核心用户角色”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

taker 是主动吃掉订单簿流动性的人。maker 是提供挂单流动性的人。LP 在 DeepBook 语境里通常指通过订单或策略提供流动性的一方，而不是 AMM LP token 的持有人。

borrower 和 liquidator 更多出现在 Margin 或闪电贷场景。DeepBookV3 spot 交易中，闪电贷是同一交易内借出和归还的 hot potato 模式；Margin 的借贷状态另属 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)。

predict trader 则属于 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict) 的业务边界。不要把这些角色混到一个对象模型里；先看包目录，再看事件和状态结构。

## 阅读补充

角色不是前端标签，而是权限和对象关系。交易者可能是 `BalanceManager` owner，也可能是持有 `TradeCap` 的被授权方；管理员通过 admin cap 管理版本、池注册和参数；Indexer/server 只读取事件和对象，不应被当成链上权限来源。

做权限设计时，把“谁签名”“谁拥有 BalanceManager”“谁持有 cap”“谁承担链下展示”拆成四列。这样能提前发现代客交易、应用授权和只读查询服务之间的边界。

## 开发要点

- 交易者身份以 `BalanceManager` owner/cap/proof 校验为准，不以前端登录态为准。
- 管理员操作要单独标注 cap 来源和可修改的协议参数。
- Indexer/server 只能解释历史和读模型，不能替代链上授权检查。

## 检查问题

- maker 和 taker 在链上一定是不同地址吗？
- 应用拿到 `TradeCap` 后能绕过 BalanceManager 校验吗？
- 管理员、应用、Indexer 三类角色的信任边界有什么不同？
