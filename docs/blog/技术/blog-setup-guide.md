---
title: Hexo + GitHub Pages | 有手就行的博客搭建指南
tags:
  - hexo
  - fluid
  - github
  - blog
createTime: 2025/10/13 21:44:50
permalink: /blog/ghp4oo4t/
---

# 前言

在开始搭建博客之前，希望你对 Git 的基本操作有一定了解，并确保你的计算机上已经安装了以下软件：

- [Node.js](https://nodejs.org/)（建议使用 LTS 版本）
- [Git](https://git-scm.com/)

可通过以下命令检查是否安装：

```bash
node -v
npm -v
git --version
```

# 创建 Github 仓库

在 GitHub 上新建一个仓库，需要特别注意的是仓库名需要为 `<your_github_username>.github.io` 。

![](/blog/blog-setup-guide/create-repo.png)

# 搭建 Hexo 博客框架

## 初始化 Hexo 项目

1. 将新建的仓库 clone 到本地（HTTPS 或 SSH 均可）：

```bash
git clone https://github.com/<your_github_username>/<your_github_username>.github.io.git
git clone git@github.com:<your_github_username>/<your_github_username>.github.io.git
```

2. 安装 Hexo：

```bash
npm install -g hexo-cli
```

3. 初始化 Hexo：

```bash
hexo init <folder>
```

由于 init 指令建站需要目标是空文件夹，所以我们可以任意指定一个文件夹，然后将其内容移动到仓库目录下。

4. 进入仓库目录：

```bash
cd <your_github_username>.github.io
```

正常情况下，项目文件夹结构应如下所示：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

## 配置主题

选择一个心仪 Hexo 主题，这里以 [Fluid](https://github.com/fluid-dev/hexo-theme-fluid) 为例。

1. 将主题仓库添加为子模块：

```bash
git submodule add https://github.com/fluid-dev/hexo-theme-fluid themes/fluid
```

2. 初始化并更新子模块：

```bash
git submodule init
git submodule update
```

> [!NOTE]
> 此处的 `<repo-url>` 必须为 HTTPS 地址。若在拉取过程中遇到连接问题，可在运行 `git submodule init` 后先将 `.gitmodules` 文件 `commit` 到本地仓库保存，然后手动更改 `.gitmodules` 中的 URL 为 SSH 形式。接着运行以下命令更新子模块：
> ```bash
> git submodule sync
> git submodule update
> ```


3. 修改仓库目录下的 `_config.yml`

```yaml
theme: fluid
```

4. 在仓库目录下创建 `_config.fluid.yml` ，将主题的 `_config.yml` 内容复制过去，原因参见 [Fluid 用户手册](https://hexo.fluid-dev.com/docs/guide/#%E8%A6%86%E7%9B%96%E9%85%8D%E7%BD%AE)。

## 本地部署

安装依赖并启动服务器：

```bash
npm install
hexo server
```

打开浏览器访问 `http://localhost:4000` 即可预览博客。

# 通过 GitHub Pages 部署

## 修改配置

修改仓库目录下的 `_config.yml` 中的 `url` ，将其更改为仓库地址：

```yaml
url: https://<your_github_username>.github.io
```

## 添加工作流文件

通常，将 `Hexo` 生成的页面通过 `GitHub Pages` 挂载后，每次提交博客都需要使用 `hexo clean && hexo generate && hexo deploy` 命令来部署，比较繁琐。本文将介绍一种借助 `GitHub Actions` 来实现自动部署的方案，每当我们提交源代码到 GitHub 存放源码的分支上，便会启动工作流，自动将构建并将静态文件 push 到发布分支，实现部署。

在仓库目录下创建 `.github/workflows/deploy.yml` 文件，内容如下：

```yaml
name: Hexo Deploy

on:
  push:
    branches: [main]

permissions:
  contents: write # 允许工作流写入仓库内容

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout # 检出代码
        uses: actions/checkout@v4

      - name: Update Submodule # 更新子模块
        run: |
          git submodule init
          git submodule update --remote

      - name: Setup Node.js # 设置Node.js环境
        uses: actions/setup-node@v4
        with:
          node-version: "22" # 指定Node.js版本（与本地一致）
          cache: "npm" # 启用 npm 缓存，减少重复下载

      - name: Install # 安装依赖
        run: |
          npm config set registry https://registry.npmmirror.com/
          npm cache clean --force
          npm install -g hexo-cli --registry=https://registry.npmmirror.com/
          npm install --registry=https://registry.npmmirror.com/

      - name: Build # 生成静态文件
        run: |
          hexo clean
          hexo generate

      - name: Deploy # 发布到gh-pages分支
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: gh-pages
```

## 启用 GitHub Pages

1. 保存并提交所有更改到 `main` 分支：

```bash
git add .
git commit -m "chore: initial commit"
git push origin main
```

若进展顺利，GitHub Actions 会自动运行工作流，并将生成的静态文件推送到远程仓库的 `gh-pages` 分支。

![](/blog/blog-setup-guide/deploy-success.png)

2. 在 GitHub 仓库页面，进入 `Settings` -> `Pages` ，在 `Source` 选项中选择 `Deploy from a branch` ，选择 `gh-pages` 分支 `/(root)` ，点击 `Save` 保存。

![](/blog/blog-setup-guide/github-pages.png)

3. 访问 `https://<your_github_username>.github.io` 即可看到你的博客上线了！

# 本地开发

在本地进行开发时，可以通过以下命令来创建一篇新文章或者新的页面：

```bash
hexo new [layout] <title>
```

同时开启本地服务器进行预览：

```bash
hexo server
```

编辑完成后，使用以下命令提交到源代码仓库：

```bash
git add .
git commit -m "<your commit message>"
git push origin main
```

> [!NOTE]
> 详细开发指南请参考[Hexo 官方文档](https://hexo.io/zh-cn/docs/)和[主题用户手册](https://hexo.fluid-dev.com/docs/guide/)。

---

[在 GitHub Pages 上部署 Hexo](https://hexo.io/zh-cn/docs/github-pages)

[Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs/start/)

[利用 GitHub Actions 自动部署 Hexo 博客](https://hexo.fluid-dev.com/posts/actions-deploy)