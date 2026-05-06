# s05 predict market ui

本目录是 Predict 市场 UI 蓝图。Predict 章节必须持续标注网络和版本状态，不能把本地源码快照误写成稳定主网能力。

## 核心页面

- market list
- market detail
- position mint form
- user positions
- LP panel
- settlement status

必须显示 oracle、到期时间、最大损失和结算规则。

## 建议命令

```bash
pnpm create vite . --template react-ts
pnpm install
pnpm add @mysten/sui decimal.js
pnpm dev
```

## 配置要求

```bash
PREDICT_VERSION_STATUS=localnet-or-testnet-explicit
PREDICT_PACKAGE_ID=0x...
PREDICT_REGISTRY_ID=0x...
PREDICT_ORACLE_ID=0x...
QUOTE_COIN_TYPE=0x...
```

## 验收

- 没有 `PREDICT_VERSION_STATUS` 时禁止提交真实交易。
- mint 前展示最大损失、fee、expiry 和 settlement 规则。
- settled market 的领取按钮必须先确认 oracle 和结算状态。
