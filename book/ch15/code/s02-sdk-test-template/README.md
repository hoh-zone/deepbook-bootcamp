# s02 sdk test template

SDK 测试关注：

- PTB 目标函数是否正确。
- type arguments 是否正确。
- object id 是否来自当前 network。
- dry run 是否通过。
- abort code 是否能映射到用户提示。

## 建议命令

```bash
pnpm init
pnpm add @mysten/sui @mysten/deepbook-v3
pnpm add -D vitest typescript tsx @types/node
pnpm exec vitest run
```

## 最小测试形状

```text
tests/
  fixtures/localnet-manifest.json
  deepbook-service.test.ts
  errors.test.ts
```

测试不应依赖真实用户私钥。交易构造可以断言 target、type arguments、objects 和 pure 参数；提交交易的测试应放到 sandbox 集成测试里。
