# s01 Sandbox Runbook

本目录记录 DeepBook Sandbox 的最小运行流程。真实源码以 `https://github.com/MystenLabs/deepbook-sandbox` 为准，本目录只保存随书步骤和验收口径。

## 启动

```bash
git clone --recurse-submodules https://github.com/MystenLabs/deepbook-sandbox.git
cd deepbook-sandbox/sandbox
pnpm install
cp .env.example .env
pnpm deploy-all
```

快速模式：

```bash
pnpm deploy-all --quick
```

## 验收

```bash
docker compose ps
curl http://localhost:9010/
curl http://localhost:3001/health
curl http://localhost:9009/manifest
```

浏览器打开：

```text
http://localhost:5173
```

## 清理

```bash
pnpm down
```

清理前保存 `deployments/localnet.json`、交易 digest 和关键日志。重新部署后地址可能变化。
