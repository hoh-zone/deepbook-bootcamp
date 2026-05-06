# ch02-07 split、join、into_balance、into_coin

[返回本章](README.md)

## 本节目标

- 掌握 Move 资产拆分、合并和对象/余额转换路径。
- 能沿“split、join、into_balance、into_coin”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

`split` 用于把一个 coin 拆成指定金额的小 coin。交易前常用于准备 gas 之外的输入资产。

`join` 用于合并同类型 coin。交易前整理碎片 coin，可以降低 PTB 输入复杂度。

`into_balance` 把 `Coin<T>` 消费成 `Balance<T>`，适合进入合约内部会计。`balance_manager.move` 的 `deposit<T>` 正是这个路径。

`into_coin` 把 `Balance<T>` 变回 `Coin<T>`，需要 `TxContext` 创建新的对象身份。`withdraw<T>` 和 `withdraw_all<T>` 使用这个路径把内部余额还给调用方。

资金流转实践：任何 `Coin<T>` 到 `Balance<T>` 的转换都应伴随事件或状态更新，否则前端和 Indexer 难以解释余额变化。DeepBook 在 `deposit` 和 `withdraw` 中都会发出 `BalanceEvent`。

## 阅读补充

资产操作的阅读顺序建议是：`split` 代表从一份余额里切出数量，`join` 代表合并回同类余额，`into_balance` 把 coin 对象变成内部余额，`into_coin(ctx)` 则需要交易上下文重新创建 coin 对象。DeepBook 的存取款和 Vault 结算会反复使用这组动作。

不要把 split/join 当作普通数字加减。它们操作的是资源，失败或遗漏会造成资源守恒问题，Move 类型系统会帮助约束，但业务逻辑仍要检查数量、资产类型和授权。

## 开发要点

- 每次 split 都要能说明切出的资产去向。
- 每次 join 都要确认资产类型一致且不会绕过费用或锁定逻辑。
- 从 `Balance<T>` 回到 `Coin<T>` 时确认函数是否需要 `TxContext`。

## 检查问题

- `into_coin(ctx)` 为什么需要 `TxContext`？
- Vault 结算中 split 和 join 分别对应哪些资产方向？
- 为什么资产操作不能等同于普通整数运算？
