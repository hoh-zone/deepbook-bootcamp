# ch10-10 strike matrix、rate limiter 和 math helper

[返回本章](README.md)

## 本节目标

- 理解 strike matrix 如何用 dense grid 记录区间库存和计算 MTM。
- 说明 rate limiter 如何约束 LP withdraw 的流动性冲击。
- 识别 math helper、SVI 和价格精度对报价实现的影响。

## 源码关联

- `packages/predict/sources/helper/strike_matrix.move`：分页 strike grid、区间库存、MTM、worst-case payout。
- `packages/predict/sources/helper/rate_limiter.move`：token bucket capacity、refill、consume。
- `packages/predict/sources/helper/math.move`、`i64.move`、`tuning_constants.move`：SVI/Black-Scholes 近似与定点数工具。

## 正文

`strike_matrix.move` 用每 512 个 strike 一页的 dense grid 记录区间边界，既支持 live curve MTM，也支持结算时 exact worst-case payout。它不是订单簿，而是 vault 对所有已 mint 区间的库存账本。

`rate_limiter.move` 是 LP withdraw 的 token bucket。默认 disabled，管理员需要设置 capacity 和 refill rate 后启用。`consume` 会拒绝超过 capacity 或当前 available 的提款；`record_deposit` 会在启用时把新增供应加入提款预算。`math.move`、`i64.move` 和 `tuning_constants.move` 提供 SVI、Black-Scholes 近似和默认参数，所有价格通常使用 `FLOAT_SCALING = 1e9`。

strike matrix 是 vault 的风险账本，不是交易撮合簿。用户 mint 并不会挂单等待成交，而是直接与 vault 定价成交；matrix 记录成交后 vault 在各个 strike 区间上的赔付曲线。

dry run 报错若来自 rate limiter，服务层可以根据 limiter 状态给出“当前可提数量”和“等待 refill”的提示；若来自 math 或 bounds，通常说明输入价格、strike 或 oracle 曲线不可用。

## 开发要点

- 风险图表用 matrix liability 曲线，不把它画成 order book depth。
- 提款 UI 显示 limiter capacity、available、refill rate 和是否 enabled。
- 所有价格计算统一使用源码精度常量，避免 JS 浮点直接参与链上金额。

## 检查问题

- strike matrix 为什么适合计算 worst-case payout？
- rate limiter 如何降低 LP 集体退出对 vault 的冲击？
- 定点数精度错误会如何影响 ask bounds 或 fee 判断？
