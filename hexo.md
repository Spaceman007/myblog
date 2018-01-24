---
title: 用hexo搭建個人博客
date: 2018-01-23 15:15:37
tags: hexo
---

# 使用

```shell
$ npm install hexo-cli -g
$ hexo init my-blog
$ cd my-blog
$ npm install
$ hexo server
```

## 1. 常用命令
命令                     | 全寫          | 含義
-------------------------|---------------|
hexo g                   | hexo generate | 生成靜態文件
hexo s                   | hexo server   | 啓動本地web服務，用於預覽
hexo d                   | hexo deploy   | 部署（到github pages等）
hexo new 'postName'      |               | 新建文章
hexo new page 'pageName' |               | 新建頁面

## 2. 配置說明（_config.yml）

```yml
language: zh-Hans
timezone: Asia/Shenzhen

# 網址
url: https://spaceman007.github.io
root: /blog

# 頭像, 將 avatar.jpg 放到 source/images 下
avatar: /images/avatar.jpg

# 代碼塊設置
highlight:
  enable: true # 使用代碼高亮
  line_number: true # 顯示行號
  auto_detect: true # 自動檢測語言

# 分頁配置
per_page: 10

# 部署配置
# 需要先安裝 hexo-deployer-git: `npm install hexo-deployer-git --save`
deploy:
  type: git
  repo:
    # 部署到coding, 曲線註釋，可同時部署
    # coding: git@git.coding.net:spaceman007/blog.git,master
    git@github.com:spaceman007.github.io.git,master
```

## 3. 使用主題

### 3.1 Next主題
**下載主題**: `$ git clone https://github.com/theme-next/hexo-theme-next`

**啓用主題**:

#### 3.1.1 編輯根路徑下的 `_config.yml`, 修改如下
```yml
# theme
# theme: false
# theme: landscape
theme: next
```

#### 3.1.2 在themes/next下，編輯 `_config.yml`, 選用 Pisces, 修改爲
```yml
#theme: Muse
#theme: Mist
theme: Pisces
```
#### 3.1.3 菜單設置
```yml
menu:
  home: /
  categories: /categories
  archives: /archives
  tags: /tags
  about: /about
  #sitemap: /sitemap
  #commonweal: /404.html
```

#### 3.1.4 文章代碼主題設置
>Next主题总共支持5种主题，默认主题是白色的normal。可以通过修改next主题下的_config.yml的highlight字段，来设置代码主题

#### 3.1.5 社交連接設置

  修改 themes/next/_config.yml的social字段：

  ```yml
  social:
    Github: https://www.github.com/spaceman007
  ```
#### 3.1.6 首頁文章摘要設置

  next默认首页文章显示所有内容，想要只显示摘要，修改next主题下_config.yml的auto_excerpt字段:

  ```yml
  auto_excerpt:
    enable: true
    length: 150 # 顯示摘要字數
  ```

#### 3.1.7 添加菜單頁面

  做完上一步会发现在首页点击菜单上的分类、归档等页面都会报错，提示没有该页面，所以需要添加各个菜单页面，定位到站点文件夹，在终端中执行新建页面指令：

  ```shell
  $ hexo new page tags
  $ hexo new page categories
  $ hexo new page about
  ```

#### 3.1.8 使用gitcommet添加評論

需要在[這裏](https://github.com/settings/developers)申請一個 OAuth App, 並將信息填入 next/_config.yml 中:
```yml
gitment:
  enable: true
  github_user: spaceman007
  github_repo: Spaceman007.github.io
  client_id: 剛申請的
  client_secret: 剛申請的
```
這時，在本地運行，評論區是禁止的，`hexo d` 發佈到github上就可以了。

# 資源
- [hexo官網](https://hexo.io/)
- [爲NexT主題添加文章月度量統計功能](sinat_2129298://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)
- [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)
- [Hexo博客搭建](https://www.wangyiting.win/2017/05/16/hexo%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA.html)
