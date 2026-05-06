# ch01-14 本章代码

[返回本章](README.md)

## 先看问题

先把“本章代码”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- `book/ch01/code/s01-env-check/README.md`
- `book/ch01/code/s02-repo-map/README.md`
- `book/ch01/code/s03-first-query/README.md`
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

本章示例目录：

- `book/ch01/code/s01-env-check`
- `book/ch01/code/s02-repo-map`
- `book/ch01/code/s03-first-query`

每个示例先以 README 形式给出命令和扩展方向。后续章节可以把这些 README 拆成可执行脚本。

## 阅读补充

本章代码都应保持“低风险、可重复、只读优先”。环境检查验证工具，repo map 验证源码路径，first query 验证链上对象入口。三者通过后，读者才具备继续进入 Move 基础和架构调用链的最低上下文。

如果要扩展本章代码，优先添加输出字段和错误提示，而不是加入复杂交易。第一章的代码目标是让读者知道自己连到了哪里、读的是哪个对象、后续应该打开哪份源码。

## 落地判断

- 示例 README 要写明输入变量和预期输出，不只给命令。
- 错误提示优先区分工具缺失、网络不通、对象不存在和类型不匹配。
- 本章代码不承担下单逻辑，避免过早引入权限和资金风险。

## 读完以后问自己

- 三个示例目录分别验证什么？
- 为什么本章代码应只读优先？
- first query 成功后，下一步应该打开哪个源码文件？
