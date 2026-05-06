# ch02-03 Sui 对象：owned、shared、immutable

[返回本章](README.md)

## 本节目标

- 区分 DeepBook 中用户对象、共享池和只读配置的访问模式。
- 能沿“Sui 对象：owned、shared、immutable”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/helper/big_vector.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/big_vector.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

owned object 由一个地址拥有，例如普通 `Coin<T>`、权限 cap、某些用户对象。它适合用户独占操作，但高频使用时可能遇到并发和锁定问题。

shared object 可被多人在交易中引用。DeepBook 的 `Pool` 和 `BalanceManager` 都是需要重点理解的共享状态。`Pool` 定义在 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，`BalanceManager` 定义在 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)。

immutable object 发布后只读。Move package 本身可按不可变依赖理解。应用构造 PTB 时必须知道对象类型，因为 owned object 需要作为输入消耗或借用，shared object 需要共享对象引用，immutable package 通过函数 target 引用。

## 阅读补充

DeepBook 中最常见的 shared object 是 `Pool` 和 Registry 相关状态，常见 owned object 是用户持有的 cap 或 `BalanceManager` 管理入口。immutable package 则提供代码和类型定义，交易调用时通过 package/module/function 定位。

判断对象类别时，除了看定义中的 `has key`，还要看创建后是 transfer 给地址、share 成共享对象，还是作为 package 发布后的不可变代码存在。不同类别决定交易是否需要所有权、是否会形成共享对象并发边界。

## 开发要点

- 写交易参数时把 owned、shared、immutable 分组列出。
- shared object 需要考虑版本和并发访问，owned object 需要考虑签名者和所有权。
- 不要把 package ID 当成普通对象余额来查询。

## 检查问题

- Pool 和 BalanceManager 在访问模式上有什么差异？
- 为什么 shared object 更容易成为并发边界？
- package ID 与普通 owned object 有什么不同？
