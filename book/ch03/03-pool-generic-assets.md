# ch03-03 Pool 的泛型资产设计

[返回本章](README.md)

## 本节目标

- 理解 base/quote 类型参数和 tick/lot/min size 约束。
- 能沿“Pool 的泛型资产设计”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/trade_params.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/trade_params.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

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

## 开发要点

- 前端和 SDK 必须固定 base/quote 顺序，展示反向价格时不要反向调用错误池。
- 创建池参数要验证 tick size、lot size、min size 的幂和整除关系。
- 对象查询时同时核对 Pool 类型和注册表记录。

## 检查问题

- 为什么 BaseAsset 与 QuoteAsset 不能相同？
- tick size 和 lot size 分别约束什么？
- 反向交易对为什么不能简单复用同一个 Pool 类型？
