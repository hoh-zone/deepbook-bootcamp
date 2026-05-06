# DeepBook 开发权威指南

本目录是正式书稿与配套代码的工作区。

- `SUMMARY.md`：全书目录。
- `OFFICIAL_DOCS_BASELINE.md`：对齐 Sui 官方 DeepBookV3、Margin、Predict 文档的事实基线。
- `LEARNING_PATH.md`：从易到难的六阶阅读路线。
- `SOURCE_DEFINITION_ATLAS.md`：关键结构体、方法和事件的内嵌定义地图。
- `MOVE_READING_GUIDE.md`：高级 Move + DeepBook 的阅读方法。
- `STYLE_GUIDE.md`：章节写作规范。
- `chXX/README.md`：每章导读、源码地图、小节目录和章节检查清单。
- `chXX/NN-title.md`：每个编号小节的独立正文。
- `chXX/code/`：每章配套代码。

本书以 Sui 官方文档作为产品和集成事实基线，以 [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0) 的 GitHub 源码作为源码讲解基础，并把 [MystenLabs/deepbook-sandbox](https://github.com/MystenLabs/deepbook-sandbox) 作为本地全栈实验环境，围绕 DeepBookV3、DeepBook Margin、DeepBook Predict、SDK 集成、Indexer、Sandbox 和生产应用开发展开。

写作目标不是复述文档，也不是堆源码链接，而是做成一本高级 Move 案例书 + DeepBook 专题书：先建立产品直觉和官方边界，再进入源码状态机，最后落到 SDK、PTB、Indexer、Server、风控和生产交付。
