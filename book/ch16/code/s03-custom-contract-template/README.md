# s03 Custom Contract Template

本目录说明如何用 sandbox 的 `example_contract` 开始写依赖 DeepBook 的 Move 合约。

## 准备

先确保已经成功执行：

```bash
cd deepbook-sandbox/sandbox
pnpm deploy-all
```

`pnpm deploy-all` 会生成 `.external-packages/` 和 `Pub.localnet.toml`，自定义合约依赖它们解析本地 DeepBook 地址。

## 复制模板

```bash
cd deepbook-sandbox
cp -r sandbox/packages/example_contract sandbox/packages/my_contract
cd sandbox/packages/my_contract
```

## 发布

```bash
sui client test-publish --build-env localnet --pubfile-path ../../Pub.localnet.toml
```

## 常见失败

- `unresolved dependency`：还没有成功跑过 `pnpm deploy-all`。
- chain id 不匹配：检查 `Move.toml` 的 `[environments] localnet`。
- 依赖地址不对：确认 `--pubfile-path ../../Pub.localnet.toml` 指向当前部署生成的文件。
- 对象参数错误：用 dashboard Deployment 页或 manifest 重新确认 package ID、pool ID 和 object ID。
