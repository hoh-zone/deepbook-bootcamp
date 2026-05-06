# ch02-02 ability 对金融对象的影响

[返回本章](README.md)

## 先建立手感

先不要把“ability 对金融对象的影响”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

`copy` 表示值可以复制。金融资产、订单所有权、权限 cap 通常不能随意 copy，否则会产生重复花费或重复授权风险。

`drop` 表示值可以被丢弃。需要强制结算或归还的资源通常不能 drop。后续闪电贷中的 hot potato 模式就依赖不能随意丢弃的资源约束。

`store` 表示值可以存入其他结构。`BalanceManager` 中把不同资产余额放入 `Bag`，依赖可存储的内部值。

`key` 表示这是链上对象。[packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 中 `BalanceManager has key, store`，说明它既是对象，也可以作为 shared object 参与 DeepBook 交易。

## 阅读补充

金融协议里 ability 不是语法装饰。资产余额、管理员 cap、交易 cap、临时 proof 是否能复制或丢弃，直接影响权限能否被滥用、资源能否被意外销毁、对象能否进入全局存储。

阅读时建议把每个 `struct has ...` 单独抄出来，旁边标注“谁创建、谁持有、能否复制、能否丢弃、能否存储”。这个表比单纯记 ability 定义更接近真实审计方法。

## Move 判断

- cap 通常不应具备 `copy`，否则权限可被复制扩散。
- proof 如果可丢弃，要确认它只是临时证明而非资产本身。
- `store` 允许嵌入其它对象或集合，要检查嵌入后谁还能访问。

## 练习问题

- 为什么 `TradeCap` 不能像普通值一样复制？
- `drop` 对临时 proof 和真实资产的风险有什么差异？
- 审计一个新 struct ability 时要问哪几个问题？
