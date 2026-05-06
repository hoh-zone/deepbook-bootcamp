# s06 audit checklist

审计材料：

- 资产清单
- 权限清单
- 关键不变量
- 交易流程图
- 错误码表
- 测试覆盖说明
- 已知风险
- 部署和升级计划

## 生成材料的建议命令

```bash
sui move test --path packages/deepbook --gas-limit 100000000000
cargo test --package deepbook-indexer
pnpm exec vitest run
pnpm exec playwright test
```

## 审计包目录

```text
audit/
  assets.md
  capabilities.md
  invariants.md
  transaction-flows.md
  error-codes.md
  test-results.md
  deployment-plan.md
```

## 验收

- 每个关键不变量都能对应到 Move 测试、SDK dry run 或运行时监控。
- 每个高权限对象都写明持有人、用途、轮换方式和泄露影响。
- 每个已知风险都有缓解措施或明确的不上线判断。
