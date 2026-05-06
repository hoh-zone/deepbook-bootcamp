# ch10-08 `range_key.move` 的 market key

[返回本章](README.md)

## 本节目标

- 掌握 `RangeKey` 的四元组语义和 sentinel 边界。
- 为二元、区间和组合市场设计不会冲突的 market key。
- 理解 market key 在 PTB、事件、前端路由和未来索引表中的作用。

## 源码关联

- `packages/predict/sources/market_key/range_key.move`：`RangeKey::new`、sentinel、settled payout。
- `packages/predict/sources/helper/constants.move`：正负无穷 sentinel 和价格精度常量。
- `book/ch10/code/s02-market-key-builder/README.md`：应用侧构造 key 的练习骨架。

## 正文

`RangeKey::new(oracle_id, expiry, lower_strike, higher_strike)` 要求 `lower < higher`，且不能直接构造 `(-inf, +inf]` 这种必胜区间。`lower == constants::neg_inf!()` 表示 DOWN sentinel，`higher == constants::pos_inf!()` 表示 UP sentinel。

应用开发中 market key 应该由 oracle ID、expiry 和 strike grid 一起派生，不能只用字符串 `"BTC-UP-100000"`。同一个 strike 在不同 oracle、不同 expiry、不同 package 部署中是不同市场。

PTB 构造时不要让前端传任意字符串 market id。服务层应从配置中的 oracle object、expiry、strike grid 和用户选择的方向派生 `RangeKey`，再把这些字段作为 move call 参数或序列化结构传入。

未来索引表可以把 `market_id` 设计成 package/network/oracle/expiry/lower/higher 的规范化字符串，但链上判断仍以 `RangeKey` 字段为准。由于 Predict Indexer 未完成，这里只是读模型建议，不是稳定 API。

## 开发要点

- 所有市场 URL、缓存 key 和事件解析都包含 oracle ID 与 expiry。
- 禁止构造 `(-inf, +inf]` 必胜区间，服务层提前拦截。
- 展示 strike 时保留链上价格精度，避免 UI 四舍五入造成错误 key。

## 检查问题

- 为什么 `BTC-UP-100000` 不是足够安全的 market id？
- `lower == neg_inf` 和 `higher == pos_inf` 分别表示哪类二元市场？
- 未来 Indexer 如果落库 market key，至少需要哪些字段？
