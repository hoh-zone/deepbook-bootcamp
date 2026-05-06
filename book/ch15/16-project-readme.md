# ch15-16 全书项目 README

[返回本章](README.md)

## 先定交付标准

这一节先看验收证据：测试、日志、监控、部署步骤、审计材料或发布前核对项必须能复现。

## 交付入口

- [book/ch13/code/s01-local-postgres/README.md](../ch13/code/s01-local-postgres/README.md)：Indexer 数据库准备。
- [book/ch14/code/s01-trading-terminal/README.md](../ch14/code/s01-trading-terminal/README.md)：应用 MVP 验收样例。
- [book/ch15/code/s05-deployment-compose/README.md](code/s05-deployment-compose/README.md)：生产拓扑模板。
- [docker](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker)：DeepBook indexer/server 容器边界。

## README 应该解决什么

全书项目 README 不是封面文案，而是读者的第一份运行手册。它至少要回答五个问题：

| 问题 | README 中的交付方式 |
| --- | --- |
| 这本书按什么顺序读 | 给出六阶路线，并说明哪些章节可以跳读。 |
| 源码以哪个版本为准 | 写明 DeepBook GitHub 快照、官方文档日期、sandbox 仓库入口。 |
| 本地如何跑 | 列出 `mdbook build`、sandbox、SDK、Indexer、Move 示例的入口命令。 |
| 示例状态如何判断 | 用“已实现、可运行、蓝图、待补齐”标注每个代码目录。 |
| 出错如何定位 | 给出 digest、checkpoint、日志、RPC、Server、PostgreSQL 的排查顺序。 |

一个合格的 README 应避免“请进入对应章节查看”这种空转句。读者第一次打开仓库时，应该能在 5 分钟内知道自己要安装什么、先运行什么、失败后看哪里。

## 推荐结构

```text
1. 书籍定位和阅读路线
2. 事实基线：官方文档、deepbookv3 源码、deepbook-sandbox
3. 环境要求：Sui CLI、Node/pnpm、Rust、Docker、PostgreSQL
4. 快速预览：mdBook 构建和本地预览
5. 本地全栈：DeepBook Sandbox
6. 章节代码运行顺序
7. 常见问题：RPC、gas、package ID、Indexer lag、Predict 版本
8. 出版和审计状态
```

## 验收标准

- README 中每条命令都能在对应目录执行，不能只写概念。
- 每个环境变量说明来源、示例值和是否允许提交到仓库。
- 每个外部链接注明用途：官方事实、源码快照、sandbox 工具或参考文档。
- 对未完成代码目录显式标注“蓝图”，不能伪装成可运行示例。

## 交付检查

- 新读者能否只凭 README 构建书稿并进入 ch16 sandbox？
- SDK、Indexer、Move 示例是否有清晰的依赖和运行边界？
- README 是否把当前仍未完成的代码 TODO 讲清楚，而不是隐藏起来？
