# ch01-11 本书项目目录规范

[返回本章](README.md)

## 本节目标

- 规范每章 sections、code 和练习记录的放置方式。
- 能沿“本书项目目录规范”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- `book/ch01`
- `book/ch02`
- `book/ch03`
- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

书稿正文放在 `book/chXX/README.md`。每章示例放在同章 `code/` 下，目录名使用 `sNN-topic`。示例 README 必须写清依赖、环境变量和运行方式。

Move 示例必须给出 `sui move build` 和 `sui move test`。TypeScript 示例优先给出 `pnpm install`、`pnpm tsx`。Indexer 示例必须说明 PostgreSQL、环境变量和 API 请求响应。

本章只创建 README 级示例，后续章节可以把 README 中的命令扩展成实际脚本。

## 阅读补充

书稿目录服务于学习路径，不服务于上游协议构建。`sections` 负责解释，`code` 负责可运行或可复现的练习，README 负责本章地图。引用上游源码时写绝对路径，是为了让读者明确自己正在读哪个仓库。

新增练习时优先放在对应章节的 `code/sXX-*` 目录，并在小节中说明它验证了哪条交易路径或源码概念。不要把一次性笔记散落在章节根目录，否则后续很难维护。

## 开发要点

- 每个小节只解释一个主题，跨章节概念用链接或后续章节入口承接。
- 练习代码目录名要能表达任务，不只写 demo。
- 引用上游源码时保留绝对路径，引用本书练习时也说明章节位置。

## 检查问题

- `sections`、`code`、章节 README 分别承担什么职责？
- 新增练习为什么要放进 `code/sXX-*`？
- 书稿仓库和 DeepBook 源码仓库为什么要分开？
