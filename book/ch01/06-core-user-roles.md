# ch01-06 核心用户角色

[返回本章](README.md)

## 先看问题

先把“核心用户角色”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

taker 是主动吃掉订单簿流动性的人。maker 是提供挂单流动性的人。LP 在 DeepBook 语境里通常指通过订单或策略提供流动性的一方，而不是 AMM LP token 的持有人。

borrower 和 liquidator 更多出现在 Margin 或闪电贷场景。DeepBookV3 spot 交易中，闪电贷是同一交易内借出和归还的 hot potato 模式；Margin 的借贷状态另属 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)。

predict trader 则属于 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict) 的业务边界。不要把这些角色混到一个对象模型里；先看包目录，再看事件和状态结构。

## 阅读补充

角色不是前端标签，而是权限和对象关系。交易者可能是 `BalanceManager` owner，也可能是持有 `TradeCap` 的被授权方；管理员通过 admin cap 管理版本、池注册和参数；Indexer/server 只读取事件和对象，不应被当成链上权限来源。

做权限设计时，把“谁签名”“谁拥有 BalanceManager”“谁持有 cap”“谁承担链下展示”拆成四列。这样能提前发现代客交易、应用授权和只读查询服务之间的边界。

## 落地判断

- 交易者身份以 `BalanceManager` owner/cap/proof 校验为准，不以前端登录态为准。
- 管理员操作要单独标注 cap 来源和可修改的协议参数。
- Indexer/server 只能解释历史和读模型，不能替代链上授权检查。

## 读完以后问自己

- maker 和 taker 在链上一定是不同地址吗？
- 应用拿到 `TradeCap` 后能绕过 BalanceManager 校验吗？
- 管理员、应用、Indexer 三类角色的信任边界有什么不同？
