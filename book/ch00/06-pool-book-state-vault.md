# ch00-06 Pool、Book、State、Vault 的分工

[返回本章](README.md)

DeepBookV3 的现货池可以从四个角色理解：`Pool` 是入口，`Book` 是订单簿，`State` 是账户和参数状态，`Vault` 是资产保管和结算。

`Pool` 位于 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)。它对外暴露用户和协议会调用的函数：创建池、下单、swap、撤单、staking、投票、claim rebates、闪电贷、burn DEEP、查询等。开发者阅读交易路径时应该先从 `Pool` 找入口，再跟进内部模块。

`Book` 位于 [packages/deepbook/sources/book](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book)。它负责订单簿本身，包括 bid、ask、order、fill、order info 和撮合队列。这里最接近传统交易所撮合引擎的核心概念：价格优先、数量扣减、剩余挂单、成交记录和取消路径。

`State` 位于 [packages/deepbook/sources/state](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state)。它不是“全局状态垃圾桶”，而是账户、历史、治理、费用参数、成交量和 rebate 计算的归属地。staking、proposal、vote、fee tier、maker rebate 等功能都需要读这里。

`Vault` 位于 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。它保存 base、quote 和 DEEP 相关余额，负责把撮合结果沉淀为资产转移。闪电贷也在这一层实现，因为借出和归还必须直接约束池子的资产保管状态。

这四个模块的边界让源码更容易读。一个下单交易大致是：`Pool` 接收参数和权限证明，`Book` 执行撮合，`State` 更新账户、费用、成交量和历史，`Vault` 处理资产流入流出，最后通过事件和返回值暴露结果。

## 本节检查

- [ ] 能说出四个模块各自负责什么。
- [ ] 能解释为什么 `Pool` 是阅读入口但不是全部逻辑。
- [ ] 能按模块描述一次下单的大致路径。
