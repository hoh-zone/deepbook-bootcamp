# ch03-10 对象借用顺序和并行边界

[返回本章](README.md)

## 本节目标

- 从函数签名判断 mutable borrow、共享对象冲突和可并行部分。
- 能沿“对象借用顺序和并行边界”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

一次下单会 mutably borrow `Pool` 和 `BalanceManager`。在 Pool 内部，`load_inner_mut` 后会修改 `book`、`state`、`vault`。所以同一个池内的订单天然串行化到同一个共享 Pool 对象上；不同池因为是不同共享对象，可以由 Sui 调度器并行处理。

BalanceManager 是另一个并行边界。两个交易如果同时修改同一个 BalanceManager，也会冲突；如果同一交易员为不同策略使用不同 BalanceManager，就能降低账户级对象冲突。

## 阅读补充

Sui 的并行执行能力要求开发者看清对象借用。对同一个 `&mut Pool` 的下单会争用同一个共享对象；不同用户的 BalanceManager 虽然是不同对象，但只要交易同时写同一个 Pool，就仍受池级撮合状态约束。

阅读函数签名时把参数分成 `&mut`、`&`、owned value、cap/proof 四类。`&mut` 决定写集合和并发冲突，`&` 多半是只读配置或时间，owned value/cap 决定权限和资源移动。

## 开发要点

- 性能分析先列 mutable shared object，不先猜测业务角色。
- 同池交易的串行边界通常在 Pool/Book 状态，跨池交易要看是否共享 Registry 或其它对象。
- 交易构造应避免无必要地传入 mutable 对象。

## 检查问题

- 为什么两个不同用户下同一个池仍可能冲突？
- `&Clock` 与 `&mut Pool` 对并发的影响有什么不同？
- 如何从函数签名快速画出借用表？
