# ch02-13 本章代码

[返回本章](README.md)

## 本节目标

- 说明 Move 基础、资产转换、共享对象和事件索引练习的用途。
- 能沿“本章代码”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- `book/ch02/code/s01-hello-move/README.md`
- `book/ch02/code/s02-coin-balance/README.md`
- `book/ch02/code/s04-event-indexing/README.md`
- [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

本章示例目录：

- `book/ch02/code/s01-hello-move`
- `book/ch02/code/s02-coin-balance`
- `book/ch02/code/s03-shared-object`
- `book/ch02/code/s04-event-indexing`

## 阅读补充

本章代码的目标是把 Move 基础语法压缩成可执行的小实验。`hello-move` 负责语法和 ability，`coin-balance` 负责资产转换，`shared-object` 负责对象访问模式，`event-indexing` 负责链上事件到链下读模型。

这些练习不需要复刻 DeepBook 全量逻辑，但每个练习都应该能指向 DeepBook 中的真实源码位置。这样读者写完小例子后，能自然迁移到 `balance_manager.move`、`pool.move` 和 `order_info.move`。

## 开发要点

- 每个练习 README 都写明对应的 DeepBook 源码文件。
- 练习代码优先验证一个 Move 概念，不混入完整交易所流程。
- 事件练习要同时给出 emit 位置和查询方式。

## 检查问题

- 四个代码练习分别对应哪些 Move 基础概念？
- 为什么小练习要绑定真实 DeepBook 文件？
- 完成本章代码后应能读懂哪些生产模块？
