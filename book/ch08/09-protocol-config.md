# ch08-09 protocol_config

[返回本章](README.md)

## 本节目标

- 理解 `ProtocolConfig` 如何把利率曲线和池级限制参数化。
- 能说明 supply cap、max utilization、min borrow、rate limit 与应用可借/可提额度之间的关系。
- 能根据 `updateInterestRates.ts` 理解维护者如何调整线上池参数。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`protocol_config.move` 定义两组配置：

- `MarginPoolConfig`：`supply_cap`、`max_utilization_rate`、`protocol_spread`、`min_borrow`、`rate_limit_capacity`、`rate_limit_refill_rate_per_ms`、`rate_limit_enabled`。
- `InterestConfig`：`base_rate`、`base_slope`、`optimal_utilization`、`excess_slope`。

利率曲线是两段式：

```text
u < optimal:  base_rate + u * base_slope
u >= optimal: base_rate + optimal * base_slope + (u - optimal) * excess_slope
```

部署脚本 [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts) 展示了管理员如何更新利率参数和池配置。例如 DEEP 池配置包含 `supplyCap`、`maxUtilizationRate`、`referralSpread`、`minBorrow`、`rateLimitCapacity` 和 `rateLimitRefillRatePerMs`。

补充说明：

两段式利率曲线让资金利用率接近或超过 optimal 时借款成本快速上升。应用侧的“可借额度”不应只展示当前 vault 余额，还要根据 max utilization 和用户风险率计算真实上限。

rate limit 保护提款速度，避免供应者在极端行情中瞬间抽干 vault。即便 supply shares 能换算出较大 amount，提款交易仍可能因为 capacity 或 refill 不足失败。

## 开发要点

- 配置面板要把池级风险参数与交易对级 PoolConfig 分开展示。
- 更新利率脚本属于维护者/管理员操作，普通用户交易流程不需要这些 cap。
- 可借额度计算同时取 vault balance、max utilization、min borrow 和用户风险约束的交集。

## 检查问题

- 当前 utilization 位于 optimal 左侧还是右侧，借款 APR 对新增借款有多敏感？
- 用户提款失败是 supply cap、rate limit 还是 vault 余额问题？
- `min_borrow` 对小额测试交易和全额还款有什么边界影响？
