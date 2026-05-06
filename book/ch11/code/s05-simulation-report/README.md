# s05 Simulation report

目标：读取 Predict simulation 输出，生成 LP 和 trader 两个视角的报告。

## 输入文件

- `results.json`
- market config
- oracle path
- user actions

## 报告字段

- vault balance
- fee reserve
- total PLP supply
- total MTM
- total max payout
- trader realized payout
- LP NAV curve
- failed transaction count

仿真报告只能用于压力测试和参数理解，不能写成收益承诺。

