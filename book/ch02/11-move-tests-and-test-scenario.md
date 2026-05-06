# ch02-11 Move 单元测试和 test_scenario

[返回本章](README.md)

## 本节目标

- 用 Move 测试复现对象创建、共享和交易调用场景。
- 能沿“Move 单元测试和 test_scenario”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- `book/ch02/code/s03-shared-object/README.md`

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

Move 单元测试通常放在同包源码或测试模块中，使用 `#[test]`、`#[test_only]` 标注。`hello_move.move` 末尾提供 `#[test_only] public fun test_helper_function(): u64`，说明测试辅助函数可以只在测试环境可见。

Sui 对象交互测试常用 `sui::test_scenario` 模拟多交易、多地址、多对象流转。ch02 示例 `book/ch02/code/s01-hello-move` 会给出最小包的 build/test 命令，后续章节再扩展到场景测试。

实践中，每次修改 Move 示例后先运行：

```bash
sui move build
sui move test
```

## 阅读补充

Move 单元测试适合验证纯逻辑、资源约束和 abort 条件；`test_scenario` 适合模拟多交易、多地址和对象所有权变化。DeepBook 完整下单依赖多个共享对象和资产状态，测试时应先拆成更小的模块行为。

写测试时不要只覆盖成功路径。金融协议文档中的检查问题最好都能变成一个 abort 测试：错误权限、错误资产类型、数量不满足 lot size、版本关闭等。

## 开发要点

- 先测小模块资源约束，再组合交易场景。
- 每个 abort 分支至少保留一个可读测试名称。
- 多地址场景要显式切换 sender，避免误判所有权。

## 检查问题

- 普通 Move 单元测试和 `test_scenario` 各适合什么问题？
- DeepBook 下单为什么不适合一开始就写成超大集成测试？
- 哪些文档检查问题可以转成 abort 测试？
