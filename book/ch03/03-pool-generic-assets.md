# ch03-03 Pool 的泛型资产设计

[返回本章](README.md)

## 先抓住结构

读“Pool 的泛型资产设计”时先画边界。一个真实协议最容易读乱的地方，不是函数太多，而是不知道 Pool、Book、State、Vault 和 BalanceManager 各自负责哪一段。

## 架构坐标

先把架构坐标钉在这些文件上。读这一节时不需要展开所有实现，只要看清对象分工、入口函数和后续会反复回来的状态边界。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

这些入口是架构地图上的锚点。先标注每个模块负责的状态，再顺着一次下单或结算回到具体函数。

## 关键定义

先看 `Pool` 的定义：

```move
public struct Pool<phantom BaseAsset, phantom QuoteAsset> has key {
    id: UID,
    inner: Versioned,
}
```

`BaseAsset` 和 `QuoteAsset` 是 phantom 类型参数。它们不作为运行时字段保存，但会进入类型系统，决定这个池到底是 `SUI/USDC`、`DEEP/SUI`，还是另一个资产对。也就是说，市场身份不是一个字符串字段，而是 Move 类型的一部分。

池创建入口也把这个设计暴露出来：

```move
public fun create_permissionless_pool<BaseAsset, QuoteAsset>(
    registry: &mut Registry,
    tick_size: u64,
    lot_size: u64,
    min_size: u64,
    creation_fee: Coin<DEEP>,
    ctx: &mut TxContext,
): ID
```

这段签名告诉开发者两件事：第一，创建池必须给出 base/quote 类型参数；第二，tick、lot、min size 是链上规则，不是前端展示规则。SDK 如果只让用户输入 `"SUI/USDC"`，最后仍然必须把它翻译成 `BaseAsset`、`QuoteAsset` 两个完整 Move type。

## 读架构

`Pool<phantom BaseAsset, phantom QuoteAsset>` 把交易对编码到类型系统里。`BaseAsset` 表示成交数量的单位，`QuoteAsset` 表示计价资产。`place_limit_order` 和 `place_market_order` 的 `quantity` 都是 base 数量；买单用 quote 支付，卖单用 base 支付。

`create_pool` 明确校验：

- `BaseAsset` 与 `QuoteAsset` 不能是同一种类型。
- `tick_size` 必须是 10 的幂。
- `lot_size >= 1000` 且是 10 的幂。
- `min_size > 0`、是 10 的幂，并且能被 `lot_size` 整除。
- 池不能同时是 whitelisted pool 和 stable pool。

这些约束不是 UI 层规则，而是链上 abort 条件。前端或 bot 在构造交易前应先读取池参数并在本地做同样校验，避免浪费 gas。

## 阅读补充

Pool 的泛型资产设计把市场身份放进类型系统：`Pool<BaseAsset, QuoteAsset>` 明确 base 和 quote 的方向，反向交易对必须作为另一个类型组合处理。价格、数量和最小下单单位再由 tick/lot/min size 约束，防止订单簿出现无法结算或无法排序的粒度。

阅读创建池逻辑时，把类型约束和数值约束分开列。类型约束回答“这是哪个市场”，数值约束回答“这个市场允许怎样的价格和数量”。

## 工程判断

- 前端和 SDK 必须固定 base/quote 顺序，展示反向价格时不要反向调用错误池。
- 创建池参数要验证 tick size、lot size、min size 的幂和整除关系。
- 对象查询时同时核对 Pool 类型和注册表记录。

## 读完以后问自己

- 为什么 BaseAsset 与 QuoteAsset 不能相同？
- tick size 和 lot size 分别约束什么？
- 反向交易对为什么不能简单复用同一个 Pool 类型？
