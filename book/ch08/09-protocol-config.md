# ch08-09 protocol_config

[返回本章](README.md)

## 先看风险边界

读“protocol_config”时先问：这一步会让账户风险变大还是变小，谁有权继续执行，失败时应该归因到价格、债务、池配置还是对象权限。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 关键定义

`ProtocolConfig` 把池级限制和利率曲线分成两组。这样维护者可以更新供给上限、提款限速，也可以单独调整利用率驱动的利率参数。

```move
public struct ProtocolConfig has copy, drop, store {
    margin_pool_config: MarginPoolConfig,
    interest_config: InterestConfig,
    extra_fields: VecMap<String, u64>,
}

public struct MarginPoolConfig has copy, drop, store {
    supply_cap: u64,
    max_utilization_rate: u64,
    protocol_spread: u64,
    min_borrow: u64,
    rate_limit_capacity: u64,
    rate_limit_refill_rate_per_ms: u64,
    rate_limit_enabled: bool,
}

public struct InterestConfig has copy, drop, store {
    base_rate: u64,
    base_slope: u64,
    optimal_utilization: u64,
    excess_slope: u64,
}
```

`max_utilization_rate` 和 `optimal_utilization` 不解决同一个问题。前者是风控硬限制，后者是利率曲线的拐点。应用计算“还能借多少”时要用硬限制；展示 APR 变化时才使用利率曲线。

## 读风险控制面

`protocol_config.move` 定义两组配置：

- `MarginPoolConfig`：`supply_cap`、`max_utilization_rate`、`protocol_spread`、`min_borrow`、`rate_limit_capacity`、`rate_limit_refill_rate_per_ms`、`rate_limit_enabled`。
- `InterestConfig`：`base_rate`、`base_slope`、`optimal_utilization`、`excess_slope`。

利率曲线是两段式：

```text
u < optimal:  base_rate + u * base_slope
u >= optimal: base_rate + optimal * base_slope + (u - optimal) * excess_slope
```

部署脚本 [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts) 展示了管理员如何更新利率参数和池配置。例如 DEEP 池配置包含 `supplyCap`、`maxUtilizationRate`、`referralSpread`、`minBorrow`、`rateLimitCapacity` 和 `rateLimitRefillRatePerMs`。

## 工程旁白

两段式利率曲线让资金利用率接近或超过 optimal 时借款成本快速上升。应用侧的“可借额度”不应只展示当前 vault 余额，还要根据 max utilization 和用户风险率计算真实上限。

rate limit 保护提款速度，避免供应者在极端行情中瞬间抽干 vault。即便 supply shares 能换算出较大 amount，提款交易仍可能因为 capacity 或 refill 不足失败。

## 风控判断

- 配置面板要把池级风险参数与交易对级 PoolConfig 分开展示。
- 更新利率脚本属于维护者/管理员操作，普通用户交易流程不需要这些 cap。
- 可借额度计算同时取 vault balance、max utilization、min borrow 和用户风险约束的交集。

## 动手检查

- 当前 utilization 位于 optimal 左侧还是右侧，借款 APR 对新增借款有多敏感？
- 用户提款失败是 supply cap、rate limit 还是 vault 余额问题？
- `min_borrow` 对小额测试交易和全额还款有什么边界影响？
