# ch11-14 本章实战：预测市场 CLI

[返回本章](README.md)

## 先跑通场景

这一节先把场景落到可执行流程：读者需要看到对象从哪里来，PTB 如何构造，交易前检查什么，失败后如何回到源码定位。

## 源码入口

- `book/ch11/code/s01-create-predict-market/` 至 `s04-settle-market/`：CLI 命令骨架。
- `packages/predict/simulations/src/runtime.ts`：PTB 构造和 dry run/执行摘要。
- `scripts/config/constants.ts`：配置集中管理形态参考。

## 从仿真到交易

CLI 的最小命令：

- `predict market create`：封装 feed id、basis bounds、oracle create、activate。
- `predict manager ensure`：派生并创建 `PredictManager`。
- `predict deposit`：把 quote coin 存入 manager。
- `predict mint --up --strike --qty`：构造 binary `RangeKey` 并 mint。
- `predict claim`：对 settled position 调 redeem。
- `predict lp supply/withdraw`：管理 PLP。

所有写交易命令都应支持 `--dry-run`，输出 PTB 摘要、预估 gas、可能 abort 和对象 ID。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- CLI 命令按 create-manager、deposit、mint、redeem、supply、withdraw、simulate 分组。
- 所有命令输出 network、package/object IDs、dry run status、digest 和错误映射。
- 默认配置指向 localnet/simulation，主网执行需要显式 `--network` 和版本确认。

## 动手检查

- CLI 哪些命令需要用户签名，哪些只读？
- mint 命令的最小参数集合是什么？
- 为什么 CLI 应默认 dry run 而不是直接提交？
