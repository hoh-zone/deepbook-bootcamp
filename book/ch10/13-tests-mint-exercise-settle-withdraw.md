# ch10-13 结合测试分析 mint、exercise、settle、withdraw

[返回本章](README.md)

## 本节目标

- 用现有测试覆盖理解 mint、redeem、settle、withdraw 的可验证路径。
- 区分已存在单元测试与迁移计划中尚未完成的端到端 Predict 测试。
- 把测试断言转化为应用 dry run 和错误解析检查项。

## 源码关联

- `packages/predict/tests/*`：fee reserve、pricing config、rate limiter、oracle cap 等当前测试。
- `packages/predict/sources/predict.move`、`vault/vault.move`、`predict_manager.move`：端到端交易路径对应实现。
- `packages/predict/simulations/src/runtime.ts`、`sim.ts`：localnet 仿真中的 PTB 执行和 gas 汇总。

## 正文

当前测试目录覆盖的是底层模块而不是完整端到端交易。`tests/accounting/fee_reserve_tests.move` 验证 fee split、默认比例、dust 留给 LP 和 zero fee；`tests/config/pricing_config_tests.move` 验证 fair price fee、min fee 和 utilization fee；`tests/helper/rate_limiter_tests.move` 验证提款限速、refill、deposit replenish 和配置错误；`tests/helper/oracle_cap_tests.move` 验证 oracle cap 注册、撤销和自撤销。

迁移文档中的 `predict_tests.move`、`predict_manager_tests.move`、`market_key_tests.move` 和 cross validation tests 仍在计划中。写生产测试时必须补齐 `create_predict -> enable_quote_asset -> create_oracle -> activate/update oracle -> create manager -> deposit -> mint -> live redeem -> settle -> settled redeem -> supply/withdraw` 的完整路径。

测试阅读时要把“当前已有覆盖”和“应补覆盖”分开。fee、pricing、limiter、oracle cap 能证明局部约束，但不能自动证明所有 mint/redeem/settle/withdraw 组合都已完成生产级验证。

应用可以把测试断言转成 dry run 预检查：余额足够、manager owner 正确、oracle fresh、range key 合法、exposure 未超、limiter 可用、settlement price 已冻结。失败后把 Move abort 映射到具体用户动作。

## 开发要点

- 测试描述不夸大为主网保障，只说明本书采用的 GitHub 源码快照覆盖面。
- 每个交易路径至少列出成功断言和一个失败断言。
- 仿真 gas 只作为 localnet 参考，不写成生产性能承诺。

## 检查问题

- 现有测试覆盖哪些模块，缺少哪些端到端场景？
- mint 失败和 withdraw 失败分别最应该 dry run 哪些条件？
- 如何把 Move abort code 映射成用户能理解的提示？
