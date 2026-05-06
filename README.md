# DeepBook Bootcamp

这是《DeepBook 开发权威指南》的 mdBook 书稿仓库。

## 预览

```bash
mdbook serve --open
```

## 构建

```bash
mdbook build
```

构建产物输出到 `target/mdbook/`，该目录不会提交到仓库。

## 目录

- `book.toml`：mdBook 配置。
- `book/SUMMARY.md`：mdBook 导航目录。
- `book/chXX/README.md`：每章导读。
- `book/chXX/NN-title.md`：每节独立正文。
- `book/chXX/code/`：每章配套示例说明。
