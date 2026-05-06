# 写作规范

## 章节结构

每章使用“章节导读 + 独立小节”的结构：

1. 本章目标
2. 本章学习阶梯
3. 源码地图
4. 小节目录，链接到 `NN-title.md`
5. 本章代码
6. Move 高阶穿插点
7. 常见错误
8. 本章检查清单
9. 进阶练习

## 从易到难的章节节奏

每章都按“场景问题 -> 最小实验 -> 源码拆解 -> 工程升级”的顺序写。源码讲解只能出现在读者已经知道它解决什么问题之后。

- **场景问题**：用交易、账户、订单、错误、风险或数据展示提出读者真实会遇到的问题。
- **最小实验**：给出可观察动作，例如查询对象、构造 PTB、dry run、查询事件或运行测试。
- **源码拆解**：沿实验背后的 Move 调用链解释对象、资源、权限、事件和 abort code。
- **工程升级**：说明 SDK、Indexer、UI、机器人、风控或生产部署应该如何利用这些结论。

每章必须写“本章学习阶梯”，明确本章从 L1 到 L5 中覆盖哪几个层级。

## 高级 Move 学习层

本书要写成“高级 Move 案例书 + DeepBook 专题书”，不能写成 API 文档堆叠。每章和重点小节都要穿插以下三类内容：

- **定义卡片**：把关键 `struct`、`public fun`、事件或 handler 的最小必要源码直接摘到书里，并解释字段、入口、状态变化和误读点。
- **Move 技巧**：解释资源、ability、泛型、对象所有权、shared object、cap、proof、hot potato、动态字段、事件和 abort code 的实际开发意义。
- **源码旁白**：告诉读者这段源码按什么顺序读，哪些变量名或模块边界容易误解，哪些字段决定状态迁移。
- **工程提醒**：把源码结论落到 SDK、PTB、dry run、Indexer、UI、错误提示、风控或生产监控。

写作时优先使用短提示框，而不是长篇抽象说明。例如：

> **Move 技巧**：看到没有 `drop` 的资源时，先找它在哪个函数中被消费；这通常就是协议安全边界。

> **源码旁白**：先读 public 入口，再读 internal helper，最后读事件和 abort code。不要从工具函数开始迷路。

## 小节写法

- 每个编号小节必须拆成独立文件：`book/chXX/NN-title.md`
- 每节标题格式：`# chXX-NN 小节标题`
- 每节开头保留返回本章链接：`[返回本章](README.md)`
- 每节至少包含：本节目标、源码关联、正文、开发要点、检查问题。
- 每节先提出场景问题或最小实验，再绑定源码，再说明开发实践。
- 源码引用必须给出 GitHub 源码，例如 `packages/deepbook/sources/pool.move`。
- 不能只给源码链接；关键结构体、方法签名和事件字段必须抽成“定义卡片”放在正文内。
- 涉及函数时写出函数名、参数意义、资金或状态变化。
- 涉及应用开发时必须补充错误处理、交易前检查和生产注意事项。
- 每节至少加入一个“Move 技巧”“源码旁白”或“工程提醒”，让内容服务学习者，而不是只罗列功能。

## 代码目录

- 每章代码放在 `book/chXX/code/`。
- 示例目录命名为 `sNN-topic`，例如 `s01-sdk-init`。
- 每个示例目录至少包含 `README.md`。
- TypeScript 示例优先使用 `pnpm` 和 `tsx`。
- Move 示例必须写清楚 `sui move build` 和 `sui move test` 命令。
- Indexer 示例必须写清楚 PostgreSQL、环境变量和 API 调用方式。

## 技术事实

- DeepBookV3 源码目录：[packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- DeepBook Margin 源码目录：[packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- DeepBook Predict 源码目录：[packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- Indexer 源码目录：[crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)
- Server 源码目录：[crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)
- DeepBookV3 闪电贷存在，入口在 `pool.move`，底层实现位于 `vault/vault.move`。

## 质量标准

- 不写空泛介绍，所有核心判断都要能落到源码、交易路径或工程实践。
- 每章至少包含一个可继续扩展的代码示例。
- 不把 DeepBook Margin 的普通借贷事件误写成 DeepBookV3 闪电贷。
- 对 Predict 标注版本和网络状态，不把迁移 TODO 写成已稳定能力。
