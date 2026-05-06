# s01-create-margin-manager

目标：创建或加载用户在某个 DeepBook Pool 上的 `MarginManager`。

输入参数：

- `--network mainnet`
- `--pool SUI_USDC`
- `--owner 0x...`
- `--create-if-missing`

推荐流程：

1. 初始化 `SuiGrpcClient` 并 `$extend(deepbook({ address }))`。
2. 根据 pool key 查询 DeepBook Pool、DeepBook Registry、MarginRegistry。
3. 查询 owner 已有 manager；找不到且允许创建时构造创建交易。
4. dry run。
5. 提交交易后解析 `MarginManagerCreatedEvent`。

运行方式建议：

```bash
pnpm tsx index.ts --network mainnet --pool SUI_USDC --create-if-missing
```

输出字段：

- `marginManagerId`
- `balanceManagerId`
- `deepbookPoolId`
- `owner`
- `created`
- `transactionDigest`

注意：创建 manager 是用户操作，不应传 `adminCap`、`marginAdminCap` 或 `marginMaintainerCap`。
