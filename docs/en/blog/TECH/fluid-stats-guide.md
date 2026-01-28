---
title: Fluid + LeanCloud | 启用网页统计配置
tags:
  - hexo
  - fluid
  - blog
  - leancloud
createTime: 2025/10/17 00:54:24
permalink: /en/article/5yq4gosl/

---

## 注册 LeanCloud 账号

1. 访问 [LeanCloud 官网](https://leancloud.cn/) 并注册一个账号。

2. 注册成功后，点击 `创建应用` 按钮，填写应用名称并选择开发版计价方案，点击 `创建` 按钮。

![](/blog/fluid-stats-guide/create-app.png)

3. 点击进入新创建的应用，点击左侧导航栏的 `数据存储` -> `结构化数据` ，点击 `创建 Class` 按钮，填写 `Class` 名称为 `Counter` ，选择 `无限制` 选项，点击 `创建` 按钮。

![](/blog/fluid-stats-guide/create-class.png)

4. 点击左侧导航栏的 `设置` -> `应用凭证` ，记录下 `AppID` 、 `AppKey` 和 `REST API 服务器地址` ，供配置使用。

![](/blog/fluid-stats-guide/credentials.png)


## 修改 Fluid 主题配置

### 配置 LeanCloud 统计

1. 打开 Hexo 博客根目录下的 `_config.fluid.yml` 文件，或者 Fluid 主题的配置文件 `_config.yml`，找到 `web_analytics` 配置项。

2. 修改 `enable` 为 `true` ，启用网页访问统计。

3. 修改 `leancloud` 配置项，填入记录的 `AppID` 、 `AppKey` 和 `REST API 服务器地址` 。

```yaml
web_analytics:
  enable: true
  leancloud:
    app_id: # AppID
    app_key: # AppKey
    server_url: # REST API 服务器地址
    ignore_local: true # 是否忽略本地访问统计，建议开启
```

### 显示页面访问统计

1. 打开 Hexo 博客根目录下的 `_config.fluid.yml` 文件，或者 Fluid 主题的配置文件 `_config.yml`，找到 `post` 配置项。

2. 修改 `meta` -> `views` 配置项，启用浏览量计数，选择统计数据来源。

```yaml
post:
  meta:
    views:
      enable: true
      source: "leancloud"
```

### 显示网站 PV、UV 统计

1. 打开 Hexo 博客根目录下的 `_config.fluid.yml` 文件，或者 Fluid 主题的配置文件 `_config.yml`，找到 `footer` 配置项。

2. 修改 `statistics` 配置项，启用 PV、UV 显示，选择统计数据来源。

```yaml
footer:
  statistics:
    enable: true
    source: "leancloud"
```

## 本地检验

1. 保存配置文件后，回到 Hexo 博客根目录，运行以下命令启动本地服务器：

```bash
hexo clean && hexo server
```

2. 打开浏览器访问 `http://localhost:4000` ，进入博客首页，查看文章标题页面底部是否显示访问统计数据。

---

[LeanCloud+Hexo 开启评论和文章阅读量](https://zhcano.github.io/LeanCloud+Hexo%E5%BC%80%E5%90%AF%E8%AF%84%E8%AE%BA%E5%92%8C%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F/)

[Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs/guide/#%E7%BD%91%E9%A1%B5%E7%BB%9F%E8%AE%A1)
