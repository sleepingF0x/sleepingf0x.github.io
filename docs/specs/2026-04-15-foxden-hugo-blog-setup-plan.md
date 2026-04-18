# Foxden Hugo Blog Setup Implementation Plan

> 状态：当前主计划。基于已确认的信息架构，在 `sleepingf0x.github.io` 中完成 Foxden 的 Hugo 建站与发布。
>
> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在新仓库 `sleepingf0x.github.io` 中搭建一个可本地预览、可自动部署到 GitHub Pages 的 Hugo 个人博客 Foxden，并落地第一版信息架构：`Home / Tech / About`。

**Architecture:** 博客使用 Hugo Extended 生成静态站点，PaperMod 主题通过 Git submodule 引入，站点源码全部维护在 `sleepingf0x.github.io` 仓库根目录。一级导航为 `Home / Tech / About`；`Tech` 是唯一技术主栏目，`Claude-Code-Guide` 作为 `Tech` 下的系列/专题承载，不作为独立仓库、一级导航或 `Tech` 子栏目。部署使用 GitHub Actions 官方 Pages 流程，push 到 `main` 后自动构建并发布到 `https://sleepingf0x.github.io`。

**Tech Stack:** Hugo Extended、PaperMod、GitHub Actions、GitHub Pages、Markdown、Git submodule

---

## Context

根据当前已确认的内容结构决策：

- `sleepingf0x.github.io` 是 Foxden 唯一主仓库与主入口
- 一级导航采用 `Home / Tech / About`
- `Tech` 是唯一技术主栏目
- `Claude-Code-Guide` 作为 `Tech` 下的系列/专题承载，可继续拆分
- 旧的 `Claude-Code-Guide` 仓库不是本计划前提，也不纳入本次实施范围

本计划覆盖新仓库 `sleepingf0x.github.io` 的初始化、信息架构落地与首次上线，不依赖保留旧仓库结构。

## Planned File Structure

**Repository root:** `sleepingf0x.github.io/`

**Create:**
- `.github/workflows/hugo.yml`
- `.gitignore`
- `.gitmodules`
- `archetypes/default.md`
- `content/_index.md`
- `content/about.md`
- `content/tech/_index.md`
- `content/tech/hello-foxden.md`
- `README.md`
- `hugo.toml`
- `static/.gitkeep`

**Create via Hugo initialization:**
- `assets/`
- `content/`
- `data/`
- `i18n/`
- `layouts/`
- `static/`
- `archetypes/`

**Create via dependency setup:**
- `themes/PaperMod/`

## External Preconditions

以下前置条件需要先完成：

1. GitHub 上已创建空仓库：`sleepingF0x/sleepingf0x.github.io`
2. 本地已有一个对应工作目录，例如：`/Users/lichenhao/Projects/sleepingf0x.github.io`
3. 该目录已执行过：

```bash
git clone https://github.com/sleepingF0x/sleepingf0x.github.io.git
```

如果目录尚未创建，则先执行：

```bash
cd /Users/lichenhao/Projects
git clone https://github.com/sleepingF0x/sleepingf0x.github.io.git
cd sleepingf0x.github.io
```

---

### Task 1: 初始化新仓库与本地站点骨架

**Files:**
- Create: `archetypes/`
- Create: `assets/`
- Create: `content/`
- Create: `data/`
- Create: `i18n/`
- Create: `layouts/`
- Create: `static/`
- Create: `hugo.toml`

- [ ] **Step 1: 确认仓库目录为空且 remote 正确**

Run:
```bash
git remote -v
git status
ls -la
```

Expected:
- `origin` 指向 `https://github.com/sleepingF0x/sleepingf0x.github.io.git`
- 工作区干净
- 根目录为空或只有 `.git` 与 GitHub 初始化文件（如 `.gitignore` / `README.md`）

- [ ] **Step 2: 确认本地已安装 Hugo Extended**

Run:
```bash
hugo version
```

Expected:
```text
hugo v0.xxx.x+extended ...
```

如果未安装：
```bash
brew install hugo
```

- [ ] **Step 3: 初始化 Hugo 项目**

Run:
```bash
hugo new site . --force
```

Expected:
- 当前仓库根目录生成 Hugo 所需目录与 `hugo.toml`
- 不报错退出

- [ ] **Step 4: 验证初始化结果**

Run:
```bash
ls
```

Expected 至少包含：
```text
archetypes
assets
content
data
i18n
layouts
static
hugo.toml
```

- [ ] **Step 5: 提交 Hugo 骨架初始化**

```bash
git add archetypes assets content data i18n layouts static hugo.toml
git commit -m "chore: initialize hugo site structure"
```

---

### Task 2: 引入 PaperMod 主题并写站点配置

**Files:**
- Create: `.gitmodules`
- Create: `themes/PaperMod/`
- Modify: `hugo.toml`

- [ ] **Step 1: 添加 PaperMod 子模块**

Run:
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Expected:
- 生成 `themes/PaperMod`
- 生成 `.gitmodules`

- [ ] **Step 2: 用完整站点配置覆盖 `hugo.toml`**

Replace `hugo.toml` with:

```toml
baseURL = 'https://sleepingf0x.github.io/'
languageCode = 'zh-cn'
defaultContentLanguage = 'zh-cn'
title = 'Foxden'
theme = 'PaperMod'
enableRobotsTXT = true

[pagination]
  pagerSize = 10

[outputs]
  home = ['HTML', 'RSS', 'JSON']

[params]
  env = 'production'
  title = 'Foxden'
  description = 'sleepingF0x 的技术笔记 · Claude Code 与工程实践'
  defaultTheme = 'auto'
  ShowReadingTime = true
  ShowShareButtons = true
  ShowPostNavLinks = true
  ShowCodeCopyButtons = true
  ShowRssButtonInSectionTermList = true
  ShowToc = true

  [params.homeInfoParams]
    Title = 'Foxden'
    Content = '记录 Claude Code、插件工具链与日常工程实践。'

  [[params.socialIcons]]
    name = 'github'
    url = 'https://github.com/sleepingF0x'

[menu]
  [[menu.main]]
    identifier = 'home'
    name = 'Home'
    url = '/'
    weight = 10

  [[menu.main]]
    identifier = 'tech'
    name = 'Tech'
    url = '/tech/'
    weight = 20

  [[menu.main]]
    identifier = 'about'
    name = 'About'
    url = '/about/'
    weight = 30
```

- [ ] **Step 3: 验证站点配置**

Run:
```bash
hugo config
```

Expected:
- 输出中包含 `title = Foxden`
- 输出中包含 `theme = PaperMod`
- 命令成功退出

- [ ] **Step 4: 提交主题与站点配置**

```bash
git add .gitmodules themes/PaperMod hugo.toml
git commit -m "feat: add PaperMod theme and site configuration"
```

---

### Task 3: 创建首页、Tech 栏目、关于页和首篇文章

**Files:**
- Modify: `archetypes/default.md`
- Create: `content/_index.md`
- Create: `content/about.md`
- Create: `content/tech/_index.md`
- Create: `content/tech/hello-foxden.md`
- Create: `static/.gitkeep`

- [ ] **Step 1: 写默认文章模板 `archetypes/default.md`**

Replace with:

```md
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
---
```

- [ ] **Step 2: 创建首页内容 `content/_index.md`**

```md
---
title: "首页"
---

欢迎来到 Foxden。

这里主要记录技术分享、工程实践和持续沉淀下来的方法。

第一版站点结构采用：

- Home
- Tech
- About
```

- [ ] **Step 3: 创建 Tech 栏目页 `content/tech/_index.md`**

```md
---
title: "Tech"
---

Tech 是 Foxden 的技术主栏目。

这里会持续整理：

- Claude Code 使用经验
- MCP、Skill 与工具链实践
- 工程流程、自动化与踩坑记录
- 逐步拆分出来的专题系列

`Claude-Code-Guide` 在这里作为一个系列/专题存在，而不是独立导航栏目。
```

- [ ] **Step 4: 创建 About 页面 `content/about.md`**

```md
---
title: "About"
---

我是 sleepingF0x。

这个博客用来记录我在 Claude Code、开发工具链和工程实践中的学习、试验与总结。

如果一篇文章能让我未来少踩一次坑，它就值得被写下来。
```

- [ ] **Step 5: 创建首篇文章 `content/tech/hello-foxden.md`**

```md
---
title: "Hello, Foxden"
date: 2026-04-15T22:30:00+08:00
draft: false
tags: ["blog", "hugo", "claude-code"]
categories: ["tech"]
series: ["Claude-Code-Guide"]
---

这是 Foxden 的第一篇文章。

当前站点先采用 `Home / Tech / About` 作为一级导航。

`Claude-Code-Guide` 会先作为 Tech 下的一个专题继续沉淀，后续再按内容发展继续拆分。
```

- [ ] **Step 6: 创建静态资源占位文件**

Create empty file:

```text
static/.gitkeep
```

- [ ] **Step 7: 本地预览站点**

Run:
```bash
hugo server -D
```

Expected:
- 首页可访问 `http://localhost:1313/`
- Tech 可访问 `http://localhost:1313/tech/`
- About 可访问 `http://localhost:1313/about/`
- 文章页可访问 `http://localhost:1313/tech/hello-foxden/`

- [ ] **Step 8: 提交初始内容页**

```bash
git add archetypes/default.md content/_index.md content/about.md content/tech/_index.md content/tech/hello-foxden.md static/.gitkeep
git commit -m "feat: add initial site structure and first tech post"
```

---

### Task 4: 配置 GitHub Pages 自动部署

**Files:**
- Create: `.github/workflows/hugo.yml`
- Create: `.gitignore`

- [ ] **Step 1: 创建 `.gitignore`**

```gitignore
public/
resources/
.hugo_build.lock
node_modules/
.DS_Store
```

- [ ] **Step 2: 创建 workflow 文件 `.github/workflows/hugo.yml`**

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.145.0
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Build with Hugo
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 3: 本地构建验证 workflow 依赖的构建命令**

Run:
```bash
hugo --minify
```

Expected:
- 构建成功
- 生成 `public/`
- 无主题与配置错误

- [ ] **Step 4: 提交部署配置**

```bash
git add .github/workflows/hugo.yml .gitignore
git commit -m "ci: add GitHub Pages deployment workflow"
```

---

### Task 5: 补充仓库说明与发布说明

**Files:**
- Create: `README.md`

- [ ] **Step 1: 写入 `README.md`**

```md
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
hugo new posts/your-post-slug.md
```

写完后把 front matter 里的 `draft: true` 改为 `false`。

## 部署

推送到 `main` 后，GitHub Actions 会自动构建并发布到 GitHub Pages。

最终地址：`https://sleepingf0x.github.io`
```

- [ ] **Step 2: 校验 README 内容**

Run:
```bash
sed -n '1,120p' README.md
```

Expected:
- 标题、开发命令、发布说明完整

- [ ] **Step 3: 提交 README**

```bash
git add README.md
git commit -m "docs: add blog readme"
```

---

### Task 6: 完成发布前验证与首次上线

**Files:**
- Verify: all created files

- [ ] **Step 1: 执行完整本地构建**

Run:
```bash
hugo --minify
```

Expected:
```text
Start building sites …
...
Total in ... ms
```

- [ ] **Step 2: 查看仓库状态**

Run:
```bash
git status
```

Expected:
- 工作区干净

- [ ] **Step 3: 推送到远端**

Run:
```bash
git push -u origin main
```

Expected:
- push 成功
- GitHub Actions 自动开始执行

- [ ] **Step 4: 启用 GitHub Pages Actions 发布**

GitHub Web UI:
- Settings
- Pages
- Build and deployment
- Source = `GitHub Actions`

- [ ] **Step 5: 验证线上页面**

访问：
- `https://sleepingf0x.github.io`
- `https://sleepingf0x.github.io/tech/`
- `https://sleepingf0x.github.io/about/`
- `https://sleepingf0x.github.io/tech/hello-foxden/`

Expected:
- 页面可访问
- 样式正常加载
- `Home / Tech / About` 菜单正常工作

---

## Verification Checklist

- [ ] GitHub 已创建 `sleepingf0x.github.io` 空仓库
- [ ] 本地 clone 目录已就绪
- [ ] Hugo Extended 已安装
- [ ] `hugo new site . --force` 初始化成功
- [ ] PaperMod 子模块引入成功
- [ ] `hugo config` 能正确读取配置
- [ ] `Home / Tech / About` 导航已落地
- [ ] `Tech` 栏目页已创建并可访问
- [ ] 首篇文章已放入 `content/tech/` 并带有 `Claude-Code-Guide` 系列标记
- [ ] `hugo server -D` 本地预览正常
- [ ] `hugo --minify` 构建成功
- [ ] GitHub Actions workflow 成功运行
- [ ] `https://sleepingf0x.github.io` 成功上线

## Scope Coverage Review

已覆盖的设计要求：
- 新仓库 `sleepingf0x.github.io` 作为唯一博客主仓库
- `Home / Tech / About` 第一版信息架构
- `Claude-Code-Guide` 作为 `Tech` 下的系列/专题落地
- Hugo + PaperMod + GitHub Actions + GitHub Pages 的完整初始化路径
- 本地开发、部署、验证与首次上线检查

未纳入本计划的内容：
- 评论系统
- 搜索
- 自定义域名
- SEO 深度优化
- 旧 `Claude-Code-Guide` 仓库的迁移与清理
- 首页专题入口区的视觉设计细化

## Not In Scope

本次不包含：
- 旧 `Claude-Code-Guide` 仓库改造、迁移或清理
- 从其他仓库自动迁移历史内容到新仓库
- 评论系统（giscus / utterances）
- 搜索能力
- 自定义域名
- Projects 页面与跨仓库导航页
- `Tech` 栏目内部更细的二级导航设计
