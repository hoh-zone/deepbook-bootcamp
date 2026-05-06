# ch02-10 abort code

[返回本章](README.md)

## 先建立手感

先不要把“abort code”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/helper/constants.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/constants.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 拆开来看

Move 中 `assert!(condition, code)` 失败会 abort。DeepBook 源码通常在模块顶部定义错误码，例如 `balance_manager.move` 的 `EInvalidOwner`、`EInvalidTrader`、`EBalanceManagerBalanceTooLow`，`pool.move` 的 `EInvalidFee`、`EInvalidTickSize`、`EMinimumQuantityOutNotMet`。

读 abort 的方法：

1. 先看交易错误中的 module 和 code。
2. 打开对应GitHub 源码文件，搜索 `const E`。
3. 找到触发 `assert!` 的函数，理解参数和状态前置条件。

生产应用不能把 abort code 原样丢给用户。应该映射成“余额不足”“订单价格不满足 tick size”“池子版本不可用”等可操作提示。

## 阅读补充

abort code 是交易失败后回到源码的索引。遇到失败时，不要只记录错误文本，要定位 module、code 和触发的 `assert!` 条件，再把它翻译成用户能理解的原因，例如余额不足、版本禁用、价格粒度错误或权限证明无效。

DeepBook 的失败通常不是单点原因。一个下单交易可能先过 Registry/Pool 版本检查，再过 tick/lot/min size，再过 BalanceManager proof 和 Vault 余额检查。阅读 abort 时要沿调用链逐层排除。

## Move 判断

- 日志里保留 module、abort code、交易 digest 和输入参数。
- 把常见 abort 映射到用户提示，但不要吞掉原始 code。
- 排查时按调用链顺序查 `assert!`，避免只看最后一个模块。

## 动手检查

- abort code 如何帮助定位 `assert!`？
- 价格粒度错误和权限错误在排查顺序上有什么不同？
- 用户提示为什么不能只写“交易失败”？
