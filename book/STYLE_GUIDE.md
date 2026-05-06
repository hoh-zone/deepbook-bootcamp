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

每个编号小节仍然必须拆成独立文件：`book/chXX/NN-title.md`。标题格式保持 `# chXX-NN 小节标题`，开头保留 `[返回本章](README.md)`，便于 mdBook 阅读和章节维护。

正文不要写成固定的“五段模板”。同一章内可以使用统一的阅读气质，但每一节都要根据内容选择节奏：

- **概念节**：先讲一个真实误解或产品问题，再给心智模型，最后指出源码入口。
- **源码节**：先给最小调用链，再摘关键定义，随后解释字段如何改变状态。
- **应用节**：先从 UI、SDK、PTB 或错误提示出发，再反推链上对象和函数。
- **数据节**：先讲读模型要回答的问题，再展开事件、表、索引和一致性。
- **练习节**：先给任务场景，再给观察点、失败路径和扩展方向。

允许使用以下标题，但不要每节都按同一组标题排列：

- `## 先看问题`
- `## 从哪段源码进入`
- `## 关键定义`
- `## 读源码`
- `## Move 旁白`
- `## 工程判断`
- `## 容易踩坑`
- `## 动手检查`
- `## 继续练习`

写法上要避免这几类模板痕迹：

- 不要每节都用“本节目标”开场；目标可以融入第一段或短提示框。
- 不要每节都列三条“能指出、能解释、能用于”的学习目标。
- 不要写“补充阅读：阅读某某时，先从……”这种生成式句子；改成具体的源码旁白。
- 不要把“开发要点”和“检查问题”写成通用三条 checklist；如果没有具体判断，就删掉。
- 不要为了形式强行加标题。短节可以用少量小标题，长节才需要更多分隔。

源码引用必须给出 GitHub 源码，例如 `packages/deepbook/sources/pool.move`。但链接不是正文，关键 `struct`、`public fun`、事件字段和 handler 映射必须抽成“关键定义”放进书里，并解释字段、入口、状态变化和误读点。

涉及函数时写出函数名、参数意义、资金或状态变化。涉及应用开发时补充错误处理、交易前检查和生产注意事项。每节至少有一个具体的 Move 旁白或工程判断，让读者读完能做出更可靠的实现决策。

## 代码目录

- 每章代码放在 `book/chXX/code/`。
- 示例目录命名为 `sNN-topic`，例如 `s01-sdk-init`。
- 每个示例目录至少包含 `README.md`。
- TypeScript 示例优先使用 `pnpm` 和 `tsx`。
- Move 示例必须写清楚 `sui move build` 和 `sui move test` 命令。
- Indexer 示例必须写清楚 PostgreSQL、环境变量和 API 调用方式。

## 技术事实

- 官方文档基线：[DeepBookV3](https://docs.sui.io/onchain-finance/deepbookv3/deepbook)、[DeepBook Margin](https://docs.sui.io/onchain-finance/deepbook-margin/)、[DeepBook Predict](https://docs.sui.io/onchain-finance/deepbook-predict/)
- DeepBookV3 源码目录：[packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- DeepBook Margin 源码目录：[packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- DeepBook Predict 源码目录：[packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- Indexer 源码目录：[crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)
- Server 源码目录：[crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)
- DeepBookV3 闪电贷存在，入口在 `pool.move`，底层实现位于 `vault/vault.move`。
- Predict 章节必须标注 Testnet / Mainnet 边界；官方文档当前把 Predict 描述为 Testnet integration surface。

## 官方文档与本书的关系

- 官方文档负责回答“当前能力是什么、公开对象和集成入口在哪里”。
- 本书负责回答“为什么这样设计、源码如何走、Move 安全边界是什么、应用如何落地”。
- 引用官方对象 ID、package ID、server URL、风险参数和网络状态时，必须标注来源和日期语境，避免把会变化的线上配置写成永久事实。
- 不能把官方文档中的 overview 直接改写成正文。每个章节必须增加源码定义、状态路径、工程判断或练习，否则达不到出版社级别。

## 质量标准

- 不写空泛介绍，所有核心判断都要能落到源码、交易路径或工程实践。
- 每章至少包含一个可继续扩展的代码示例。
- 不把 DeepBook Margin 的普通借贷事件误写成 DeepBookV3 闪电贷。
- 对 Predict 标注版本和网络状态，不把迁移 TODO 写成已稳定能力。
