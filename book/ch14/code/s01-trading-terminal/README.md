# s01 trading terminal

本目录是交易终端 MVP 的实现蓝图。它对应 ch14 的产品层，不直接托管用户私钥；前端构造交易，钱包签名，后端或 DeepBook Server 只提供读模型。

## MVP 模块

- pool selector
- orderbook
- recent trades
- balance manager panel
- order form
- open orders
- fills history

交易确认需要同时监听交易 digest 和 Indexer 数据刷新。

## 建议文件结构

```text
src/
  config/localnet.ts
  components/OrderBook.tsx
  components/OrderForm.tsx
  components/BalanceManagerPanel.tsx
  services/deepbook.ts
  services/deepbookServer.ts
```

## 启动命令

```bash
pnpm create vite . --template react-ts
pnpm install
pnpm add @mysten/dapp-kit @mysten/sui @mysten/deepbook-v3
pnpm dev
```

## 验收

- 钱包切到 localnet 后能读取 `http://localhost:9009/manifest`。
- 下单后页面记录 transaction digest。
- 订单状态先以链上确认作为成功证据，再等待 Indexer/Server 刷新。
- Indexer 落后时 UI 显示 pending，不把空列表误判为订单不存在。
