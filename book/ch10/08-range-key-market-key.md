# ch10-08 `range_key.move` 的 market key

[返回本章](README.md)

## 先看市场问题

Predict 市场最怕“看起来一样”的 key。`BTC-UP-100000` 这样的字符串在 UI 里很直观，但链上并不知道它属于哪个 oracle、哪个到期日、哪个部署环境，也不知道边界价格怎么处理。

`RangeKey` 解决的是这个问题：把市场身份压成 oracle、expiry、lower strike、higher strike 四个字段。后面的 mint、vault 记账、settlement 和 indexer 都应该围绕这四个字段展开。

## 源码入口

- `packages/predict/sources/market_key/range_key.move`：`RangeKey::new`、sentinel 校验和 settled payout。
- `packages/predict/sources/helper/constants.move`：正负无穷 sentinel 和价格精度常量。
- `book/ch10/code/s02-market-key-builder/README.md`：应用侧构造 key 的练习骨架。

## 关键定义

`RangeKey` 用四个字段唯一表达一个区间市场。它不是 UI 字符串，也不是数据库自增 id，而是 PredictManager 和 Vault 都能验证的链上 key。

```move
public struct RangeKey has copy, drop, store {
    oracle_id: ID,
    expiry: u64,
    lower_strike: u64,
    higher_strike: u64,
}

public fun new(
    oracle_id: ID,
    expiry: u64,
    lower_strike: u64,
    higher_strike: u64,
): RangeKey {
    assert!(lower_strike < higher_strike, EInvalidStrikes);
    assert!(
        !(lower_strike == constants::neg_inf!() && higher_strike == constants::pos_inf!()),
        EInvalidStrikes,
    );
    RangeKey { oracle_id, expiry, lower_strike, higher_strike }
}

public(package) fun settled_payout(key: &RangeKey, settlement: u64, quantity: u64): u64 {
    settled_range_payout(settlement, key.lower_strike, key.higher_strike, quantity)
}
```

`settled_payout` 把区间定义写死为 `(lower, higher]`。这对产品设计很重要：边界价格恰好等于 lower 时不支付，恰好等于 higher 时支付。前端、合约测试和 indexer 都必须使用同一套边界语义。

## 读市场参数

`RangeKey::new` 的两条校验都和产品语义直接相关。第一条要求 `lower < higher`，避免无效区间；第二条禁止 `(-inf, +inf]`，避免构造必胜市场。`lower == constants::neg_inf!()` 表示 DOWN sentinel，`higher == constants::pos_inf!()` 表示 UP sentinel。

应用开发中 market key 应该由 oracle ID、expiry 和 strike grid 一起派生，不能只用字符串 `"BTC-UP-100000"`。同一个 strike 在不同 oracle、不同 expiry、不同 package 部署中是不同市场。

PTB 构造时不要让前端传任意字符串 market id。服务层应从配置中的 oracle object、expiry、strike grid 和用户选择的方向派生 `RangeKey`，再把这些字段作为 move call 参数或序列化结构传入。

未来索引表可以把 `market_id` 设计成 package/network/oracle/expiry/lower/higher 的规范化字符串，但链上判断仍以 `RangeKey` 字段为准。由于 Predict Indexer 未完成，这里只是读模型建议，不是稳定 API。

## Predict 边界判断

- 市场 URL、缓存 key 和事件解析至少包含 network/package、oracle ID、expiry、lower、higher。
- 服务层不要让前端传任意 market id 字符串；应由配置和用户选择派生 `RangeKey`。
- strike 展示可以格式化，提交参数必须保留链上整数精度。

## 动手检查

- 为什么 `BTC-UP-100000` 不是足够安全的 market id？
- `lower == neg_inf` 和 `higher == pos_inf` 分别表示哪类二元市场？
- 未来 Indexer 如果落库 market key，至少需要哪些字段？
