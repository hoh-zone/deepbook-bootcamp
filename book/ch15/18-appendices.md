# ch15-18 附录

[返回本章](README.md)

## 先定交付标准

这一节先看验收证据：测试、日志、监控、部署步骤、审计材料或发布前核对项必须能复现。

## 附录入口

- [crates/schema/src/schema.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/schema/src/schema.rs)：表结构和 Diesel schema。
- [crates/server/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/README.md)：REST API 与服务运行边界。
- [packages/deepbook/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/tests)：Spot 测试 fixture。
- [packages/deepbook_margin/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/tests)：Margin 测试 fixture。

## 附录应该怎么用

附录服务查阅，不承载新概念。正文中第一次讲清楚对象、资金路径和风险边界；附录负责让读者快速反查名称、路径、字段和命令。

建议最终拆成九类：

| 附录 | 内容 |
| --- | --- |
| A 函数索引 | `place_limit_order`、`borrow_flashloan_base`、Margin/Predict 入口等。 |
| B 事件索引 | 订单、成交、余额、闪电贷、Margin 借还、Predict mint/settle。 |
| C 结构体索引 | `Pool`、`Book`、`Order`、`BalanceManager`、`Vault`、`MarginManager`。 |
| D 错误码索引 | abort code、触发条件、用户提示、dry run 定位方式。 |
| E 表结构索引 | Indexer 表、主键、查询模式、checkpoint 字段。 |
| F SDK 片段 | 初始化、BalanceManager、下单、撤单、dry run、错误解析。 |
| G 命令速查 | `mdbook`、`sui move`、`pnpm`、`docker compose`、`psql`。 |
| H 术语中英对照 | pool、vault、fill、rebate、checkpoint、hot potato。 |
| I 事实核对清单 | 官方文档、源码快照、sandbox README、网络状态和版本边界。 |

## 收录规则

- 只收正文已经解释过的概念，不在附录里第一次引入关键机制。
- 每个函数或事件都标注源码路径、章节来源和应用侧用途。
- 表结构要写主键、常用过滤条件和一致性风险，不能只列字段。
- 错误码要写用户看到什么、开发者查什么、是否可重试。

## 交付检查

- 读者能否从一个错误码反查到源码、dry run 和用户提示？
- 读者能否从一个事件名反查到 Indexer 表和 Server API？
- 附录是否区分 DeepBookV3、Margin、Predict、sandbox 和本书示例？
