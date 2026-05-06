# ch12-08 限价单封装

[返回本章](README.md)

## 本节目标

- 封装限价单 PTB，覆盖价格、数量、方向、过期和 self-matching 选项。
- 绑定 `pool.move` 入口或 DeepBook SDK builder。
- 在 dry run 后解析余额、cap、lot size 和版本错误。

## 源码关联

- `scripts/transactions/deepbookMarketMaker.ts`：限价单 SDK 调用示例。
- `packages/deepbook/sources/pool.move`：`place_limit_order` 等入口。
- `scripts/utils/utils.ts`：dry run 与 gas 设置。

## 正文

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

## 开发要点

- 限价单请求验证 poolKey、price、quantity、isBid、expiration、clientOrderId。
- PTB target 或 SDK builder 调用要能在单元测试中断言。
- dry run 后检查订单对象变化、余额锁定和 Move abort。

## 检查问题

- 限价单服务方法最少需要哪些参数？
- price/quantity 精度错误会导致哪类失败？
- 如何把 order already expired 或余额不足提示给用户？
