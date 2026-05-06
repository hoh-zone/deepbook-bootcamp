# DeepBook 书稿质量审查记录

审查日期：2026-05-06

本文件记录本轮全文质量审查的结论。它不是读者正文，而是后续继续打磨书稿时的编辑基线。

## 本轮发现

1. **入口章节仍有模板痕迹**

   ch01-ch03 中残留 “本节重点对照以下源码或后续阅读入口”“阅读时先从标题对应的入口文件开始” 这类句式。它们能导航，但不像正式书稿。已批量改成更自然的“建立定位能力”和“回到入口核对”的表达。

2. **应用章节部分小节太像功能说明**

   ch11、ch12、ch14、ch15 中有短小节只说明要做什么，没有充分解释为什么要这样做、失败时如何定位、用户应看到什么。已优先重写 Predict 信息架构、Predict deposit、PredictService、应用构建路线、Margin 仪表盘、Indexer snapshot、出版标准和章节验收标准。

3. **后段章节容易缺少证据意识**

   测试、部署、出版交付章节不能只写“要有测试、要有监控”。出版社级别的标准应落到可复现证据：命令、fixture、snapshot、digest、checkpoint、日志、指标、运行手册和风险披露。已重写 ch15 相关入口。

4. **Predict 必须持续标注版本边界**

   官方文档当前把 Predict 描述为 Testnet integration surface。书中涉及 package/object IDs、server、indexer、SDK、entry points 时，必须区分官方 Testnet 集成面、本地源码快照和未来 Mainnet 部署前可能变化的部分。

## 已完成修订

- 新增 `OFFICIAL_DOCS_BASELINE.md`，把官方 DeepBookV3、Margin、Predict 文档作为事实基线。
- 更新 `STYLE_GUIDE.md`，明确官方文档、源码定义、状态路径、工程判断和风险边界的关系。
- 改写 ch01/ch15 章导读，使入口章和交付章更像正式书稿。
- 批量移除 ch01-ch03 早期模板句。
- 批量降低 ch11/ch12/ch14/ch15 的机械开头。
- 手工重写一组代表性粗糙小节：
  - `book/ch11/01-predict-app-information-architecture.md`
  - `book/ch11/04-deposit-quote-asset.md`
  - `book/ch12/20-predict-service.md`
  - `book/ch14/01-app-build-route.md`
  - `book/ch14/17-margin-dashboard-mvp.md`
  - `book/ch15/04-indexer-snapshot-tests.md`
  - `book/ch15/14-publishing-quality-standard.md`
  - `book/ch15/15-chapter-acceptance-standard.md`

## 后续仍需继续打磨

- ch10-ch12 中少数 Predict 应用小节仍偏短，需要继续补充“用户看到什么、交易前检查什么、失败如何定位”。
- ch12 SDK 小节需要逐步补齐可运行依赖、环境变量和命令。
- ch13 Indexer 小节需要继续补 PostgreSQL schema、API 请求和响应样例。
- ch14 MVP 小节需要补每个产品的最小数据模型和降级策略。
- ch15 需要补完整运行手册和出版前事实核对清单。

## 出版级判定标准

一个小节可以进入出版级状态，至少要满足：

- 有明确读者场景或真实工程问题。
- 有官方文档或源码事实依据。
- 关键结构体、函数、事件或 handler 不只给链接，必要时直接摘入正文。
- 解释对象、资金、事件或读模型如何变化。
- 写清应用侧如何用：SDK、PTB、dry run、Indexer、UI、监控或部署。
- 明确失败路径和风险边界，尤其是 Margin、Predict 和数据延迟。
