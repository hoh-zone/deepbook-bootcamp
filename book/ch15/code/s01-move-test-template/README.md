# s01 move test template

Move 测试模板应包含：

- 正常路径
- expected failure
- 边界值
- 权限错误
- 资金不足
- 错误资产类型

命令示例：

```bash
sui move test --path packages/deepbook --gas-limit 100000000000
```

