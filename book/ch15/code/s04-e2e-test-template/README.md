# s04 e2e test template

端到端测试步骤：

1. 连接钱包。
2. 构造交易。
3. 签名提交。
4. 等待链上确认。
5. 等待 Indexer 确认。
6. 校验 UI 状态。

## 建议命令

```bash
pnpm add -D @playwright/test
pnpm exec playwright install
pnpm exec playwright test
```

## 测试边界

- E2E 不直接证明合约安全，只证明用户路径和状态机可用。
- 每个写交易都要保存 digest。
- UI 断言要同时处理链上已成功但 Indexer 尚未刷新的 pending 状态。
