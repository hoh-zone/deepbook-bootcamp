# ch01-14 本章代码

[返回本章](README.md)

## 本节目标

- 说明环境检查、仓库地图和首次查询示例如何支撑第一章。
- 能沿“本章代码”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- `book/ch01/code/s01-env-check/README.md`
- `book/ch01/code/s02-repo-map/README.md`
- `book/ch01/code/s03-first-query/README.md`
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

本章示例目录：

- `book/ch01/code/s01-env-check`
- `book/ch01/code/s02-repo-map`
- `book/ch01/code/s03-first-query`

每个示例先以 README 形式给出命令和扩展方向。后续章节可以把这些 README 拆成可执行脚本。

## 阅读补充

本章代码都应保持“低风险、可重复、只读优先”。环境检查验证工具，repo map 验证源码路径，first query 验证链上对象入口。三者通过后，读者才具备继续进入 Move 基础和架构调用链的最低上下文。

如果要扩展本章代码，优先添加输出字段和错误提示，而不是加入复杂交易。第一章的代码目标是让读者知道自己连到了哪里、读的是哪个对象、后续应该打开哪份源码。

## 开发要点

- 示例 README 要写明输入变量和预期输出，不只给命令。
- 错误提示优先区分工具缺失、网络不通、对象不存在和类型不匹配。
- 本章代码不承担下单逻辑，避免过早引入权限和资金风险。

## 检查问题

- 三个示例目录分别验证什么？
- 为什么本章代码应只读优先？
- first query 成功后，下一步应该打开哪个源码文件？
