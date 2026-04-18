# Foxden

个人博客，第一版信息架构采用 `Home / Tech / About`。技术内容统一收敛到 `Tech`，其中 `Claude-Code-Guide` 作为一个系列/专题继续沉淀。

## 本地开发

启动本地开发服务器：

```bash
hugo server -D
```

访问：`http://localhost:1313`

## 新建文章

```bash
hugo new tech/your-post-slug.md
```

写完后把 front matter 里的 `draft: true` 改为 `false`。

如果文章属于某个专题，可以补充 `series` 字段，例如：

```toml
series = ["Claude-Code-Guide"]
```

## 部署

推送到 `main` 后，GitHub Actions 会自动构建并发布到 GitHub Pages。

最终地址：`https://sleepingf0x.github.io`
