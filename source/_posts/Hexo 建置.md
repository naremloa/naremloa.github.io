---
title: Hexo建置
id: hexo_setup
date: 2020-05-21 16:37:25
categories: Hexo
description: 該網站的建立過程記錄。包括建立流程，配置設定，相關套件使用和參考文章。
tags:
  - Hexo
---

> [hexo-cli@3.1.0](https://www.npmjs.com/package/hexo-cli) + [hexo-theme-next@v8.0.0-rc.2](https://github.com/next-theme/hexo-theme-next/releases/tag/v8.0.0-rc.2)

## 專案建立

參照[官網](https://hexo.io/zh-tw/docs/setup)的介紹在本地將專案建起，並完成[NexT](https://github.com/next-theme/hexo-theme-next)的安裝介紹。

Hexo 的主題這塊會有個 _submodule_ 問題，如果要同時兼顧客製化修改和持續性更新，有一篇[「暴力而不失優雅的升級 Hexo 主題」](https://zhangnai.xin/2018/11/11/hexo-theme-upgrade/)有討論相關內容，這邊暫不贅述。

## 客製化主題

客製化主要是從主題的`_config.yml`和其他模板檔案下手。方式有很多種(前面介紹的文章)，但在下手修改時，建議先考慮到主題後續更新的問題。

如果沒有需要過多的客製化，推薦採用`data files`方式。相關[issue](https://github.com/iissnan/hexo-theme-next/issues/328)。

這邊純紀錄改動過的設定：

```yml
# next.yml
# 客製化檔案路徑
custom_file_path:
  # 客製化head標籤的內容
  # 這邊主要是解決google search console的驗證問題
  head: source/_data/head.njk

# 各種icon
favicon:
  small: <favicon 16x16>
  medium: <favicon 32x32>
  apple_touch_icon: <apple_touch_icon>
  safari_pinned_tab: <safari_pinned_tab>

# 底部條顯示設定
footer:
  # 底部網站建立時間
  since: <setup date>
  # 底部icon
  icon:
    name: fas fa-terminal
    animated: false
    color: "#999999"
  # 底部是否顯示 Powered by Hexo & NexT
  powered: false

# 創用CC設定
creative_commons:
  license: by-nc-nd
  sidebar: true
  post: true
  language: "zh-TW"

# NexT主題
scheme: Pisces

# 暗黑模式
darkmode: true

# 側欄菜單設定
menu:
  home: / || fa fa-home
  tags: /tags/ || fa fa-tags
  archives: /archives/ || fa fa-archive
menu_settings:
  icons: true
  badges: false

# 頭像設定
avatar:
  url: <your avatar>
  rounded: true
  rotated: false

# 社交鏈接
social:
  GitHub: <link> || fab fa-github
  E-Mail: mailto:<mail> || fa fa-envelope
  Steam: <link> || fab fa-steam

# 控制標籤，用icon替代#
tag_icon: true

# 控制是否在底部顯示前一篇或後一篇文章
post_navigation: false

# 返回頂部按鈕設置
back2top:
  enable: true
  sidebar: true
  scrollpercent: true

# 設置書籤
bookmark:
  enable: true
  color: "#222"
  save: auto

# disqus設置
disqus:
  enable: true
  shortname: <your shortname>

# searchdb設置
local_search:
  enable: true

# _config.yml
# site
title: <標題>
subtitle: <副標題>
author: naremloa
language: "zh-TW"

# URL
url: <your site url>
# 文章永久鏈接格式
permalink: :year/:month/:day/:id/

# 擴充套件
Plugins:
  - hexo-generator-sitemap

# 主題名稱
theme: next

# Hexo部署指令的相關設定
deploy:
  type: git
  repository: <my repository url>
  branch: master
```

## 套件介紹

### hexo-generator-sitemap

會在`hexo generate`的過程中，順帶生成一份`sitemap.xml`。可以給 _google search console_ 做搜索優化用。

注意，在 _google search console_ 中提交 _sitemap_ 時，填寫的網址要帶上`.xml`，如`<your url>/sitemap.xml`，不然讀取不到。

### hexo-deployer-git

配合`hexo deploy`使用，簡化部署流程。如果不走支持的部署方式，就不需要該套件了。

### hexo-generator-searchdb

提供站內搜索。支持模糊匹配，兩個關鍵字中間要有個空格隔開。

## 參考鏈接

1. [Hexo](https://hexo.io/zh-cn/)
2. [NexT](https://theme-next.js.org/)
3. [暴力而不失優雅的升級 Hexo 主題](https://zhangnai.xin/2018/11/11/hexo-theme-upgrade/)
4. [Theme configurations using Hexo data files](https://github.com/iissnan/hexo-theme-next/issues/328)
5. [Hexo 系列文章](https://blog.typeart.cc/hexoSeries/)
6. [數據文件](https://tding.top/docs/getting-started/data-files.html)
