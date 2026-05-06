# ch08-08 margin_state 与利息累计

[返回本章](README.md)

## 先看风险边界

这里先把问题放到风险控制面里。“margin_state 与利息累计”不是在 DeepBook 旁边加一层 UI，而是把 registry、manager、oracle、borrow/supply state 接进同一条链上风控路径。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/margin_pool/protocol_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/protocol_config.move)
- `book/ch08/code/s03-interest-accrual/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 关键定义

`State` 用 amount 和 shares 分离“池总量”和“用户份额”。利息改变 `total_supply` / `total_borrow`，但不会直接改变每个用户持有的 shares。

```move
public struct State has drop, store {
    total_supply: u64,
    total_borrow: u64,
    supply_shares: u64,
    borrow_shares: u64,
    last_update_timestamp: u64,
    extra_fields: VecMap<String, u64>,
}

public(package) fun increase_supply(
    self: &mut State,
    config: &ProtocolConfig,
    amount: u64,
    clock: &Clock,
): (u64, u64) {
    let protocol_fees = self.update(config, clock);
    let ratio = self.supply_ratio();
    let shares = math::div(amount, ratio);
    self.supply_shares = self.supply_shares + shares;
    self.total_supply = self.total_supply + amount;

    (shares, protocol_fees)
}
```

供应 shares 使用向下取整，借款 shares 在增加债务时使用向上取整，这是借贷协议里常见的保守精度处理：不能让借款人因为整数舍入少承担债务。读 MarginPool 代码时，先找每个入口是否调用 `update`，再看 shares 和 amount 的换算方向。

## 读风险控制面

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

## 工程旁白

利息累计是懒更新模型：只有供应、提款、借款、还款等操作进入 `update` 时，才按 `last_update_timestamp` 到当前时间计算一次。用户 shares 静止，池的 `total_borrow` 和 `total_supply` 改变，导致 shares 换算出来的 amount 改变。

协议费用来自利息而不是本金。`protocol_spread` 从借款人产生的 interest 中划出一部分进入 fee 逻辑，其余体现在供应侧收益中。应用估算 APR 时应基于 utilization 和利率曲线，而不是把过去收益线性外推。

## 风控判断

- 每次展示 debt amount 前用最新 state 换算 borrow shares，不要缓存借款时 amount。
- 模拟还款时注意 round up/round down，避免还款金额换算成 0 shares。
- 利率图要标出 optimal utilization，因为超过后 excess slope 会显著抬高借款成本。

## 动手检查

- 多久没有触发 `update`，这会如何影响下一次操作中的利息跳变？
- borrow shares 和 supply shares 分别在什么场景增加或减少？
- protocol fees 从哪部分利息中扣除，是否影响供应者收益展示？
