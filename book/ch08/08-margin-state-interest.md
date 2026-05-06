# ch08-08 margin_state 与利息累计

[返回本章](README.md)

## 本节目标

- 读懂 `margin_state::State` 如何用 total supply/borrow 与 shares 累计利息。
- 能解释 utilization 驱动利率、protocol spread 扣费和 elapsed time 对 debt amount 的影响。
- 能说明为什么 borrow shares 使用 round up，供应和借款 shares 不会每秒改变。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- `book/ch08/code/s03-interest-accrual/README.md`

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

`margin_state::State` 保存：

- `total_supply`
- `total_borrow`
- `supply_shares`
- `borrow_shares`
- `last_update_timestamp`

每次 `increase_supply`、`decrease_supply_shares`、`increase_borrow`、`decrease_borrow_shares` 都先调用 `update`。`update` 根据当前利用率和经过时间计算利息：

```text
utilization_rate = total_borrow / total_supply
interest_rate = piecewise_rate(utilization_rate)
interest = total_borrow * interest_rate * elapsed_ms / year_ms
protocol_fees = interest * protocol_spread
total_supply += interest - protocol_fees
total_borrow += interest
```

supply shares 使用 `math::div(amount, supply_ratio)`，borrow shares 使用 `math::div_round_up(amount, borrow_ratio)`。借款 round up 是为了避免借款人用精度误差少记债务。

补充说明：

利息累计是懒更新模型：只有供应、提款、借款、还款等操作进入 `update` 时，才按 `last_update_timestamp` 到当前时间计算一次。用户 shares 静止，池的 `total_borrow` 和 `total_supply` 改变，导致 shares 换算出来的 amount 改变。

协议费用来自利息而不是本金。`protocol_spread` 从借款人产生的 interest 中划出一部分进入 fee 逻辑，其余体现在供应侧收益中。应用估算 APR 时应基于 utilization 和利率曲线，而不是把过去收益线性外推。

## 开发要点

- 每次展示 debt amount 前用最新 state 换算 borrow shares，不要缓存借款时 amount。
- 模拟还款时注意 round up/round down，避免还款金额换算成 0 shares。
- 利率图要标出 optimal utilization，因为超过后 excess slope 会显著抬高借款成本。

## 检查问题

- 多久没有触发 `update`，这会如何影响下一次操作中的利息跳变？
- borrow shares 和 supply shares 分别在什么场景增加或减少？
- protocol fees 从哪部分利息中扣除，是否影响供应者收益展示？
