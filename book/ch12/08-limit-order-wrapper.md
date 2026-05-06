# ch12-08 限价单封装

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `scripts/transactions/deepbookMarketMaker.ts`：限价单 SDK 调用示例。
- `packages/deepbook/sources/pool.move`：`place_limit_order` 等入口。
- `scripts/utils/utils.ts`：dry run 与 gas 设置。

## 关键定义

SDK wrapper 最终要构造这个 Move call：

```move
public fun place_limit_order<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
    client_order_id: u64,
    order_type: u8,
    self_matching_option: u8,
    price: u64,
    quantity: u64,
    is_bid: bool,
    pay_with_deep: bool,
    expire_timestamp: u64,
    clock: &Clock,
    ctx: &TxContext,
): OrderInfo
```

因此 TypeScript wrapper 的职责不是隐藏这些参数，而是把用户输入稳定转换成它们：

- `poolKey` -> `Pool<BaseAsset, QuoteAsset>` 和 type arguments。
- `balanceManagerKey` -> `BalanceManager` 对象和 `TradeProof` 生成方式。
- decimal `price/quantity` -> 链上整数，并校验 tick/lot。
- `orderType/selfMatchingOption` -> DeepBook 常量。
- `expire_timestamp` -> 毫秒时间戳或默认过期策略。

## SDK 读法

限价单对应 `packages/deepbook/sources/pool.move` 的 `place_limit_order`。SDK 层不要直接暴露 Move 参数顺序，而是暴露领域参数：

```ts
type LimitOrderInput = {
  poolKey: string;
  balanceManagerKey: string;
  clientOrderId: string;
  price: number;
  quantity: number;
  isBid: boolean;
  orderType?: "NO_RESTRICTION" | "IMMEDIATE_OR_CANCEL" | "POST_ONLY";
  selfMatchingOption?: "ALLOW" | "CANCEL_TAKER" | "CANCEL_MAKER";
  payWithDeep?: boolean;
};
```

提交前要调用 `can_place_limit_order` 或做 dry run，错误时把价格精度、数量精度、余额不足、pool 不存在和版本不允许区分开。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 限价单请求验证 poolKey、price、quantity、isBid、expiration、clientOrderId。
- PTB target 或 SDK builder 调用要能在单元测试中断言。
- dry run 后检查订单对象变化、余额锁定和 Move abort。

## 动手检查

- 限价单服务方法最少需要哪些参数？
- price/quantity 精度错误会导致哪类失败？
- 如何把 order already expired 或余额不足提示给用户？
