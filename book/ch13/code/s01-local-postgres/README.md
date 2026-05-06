# s01 local postgres

目标：准备 DeepBook Indexer 本地数据库。

```bash
createdb deepbook
export DATABASE_URL="postgresql://postgres:postgrespw@localhost:5432/deepbook"
```

如果使用 Docker，可以固定数据目录，避免容器重启后丢失索引数据。生产环境不要把数据库和应用容器生命周期绑定在一起。

