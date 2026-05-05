---
title: 从零搭建我的 Hexo + Aero 个人博客
date: 2026-05-05 23:50:00
tags:
  - Hexo
  - 博客搭建
  - Cloudflare Pages
  - Waline
categories:
  - 博客搭建
---

这篇文章记录我从零搭建个人博客的完整过程。

最后得到的效果是：

- 用 Hexo 生成静态博客
- 使用 `hexo-theme-aero` 作为主题
- 用 GitHub 保存博客源码
- 用 Cloudflare Pages 自动部署博客
- 用 Waline 做留言板和文章评论
- 用 Vercel + Neon 部署 Waline 服务端
- 用自定义域名访问博客和评论服务
- 用阿里云 OSS 托管音乐文件

如果以后要重新搭一个类似的站点，可以直接照着这篇文章走一遍。

## 1. 选择博客方案

一开始我在两个项目之间做选择：

- `hexo-theme-aero`
- `flare-stack-blog`

最后选择了 `hexo-theme-aero`。

原因很简单：我现在更需要一个能快速上线、方便写 Markdown 的个人博客，而不是一个完整的内容管理系统。

`hexo-theme-aero` 是 Hexo 主题，部署成本低，写文章也直接。`flare-stack-blog` 功能更完整，但是需要 Cloudflare Workers、数据库、后台、鉴权等配置，对刚开始搭博客来说有点重。

## 2. 初始化 Hexo 项目

我的博客目录是：

```text
D:\code\blog
```

在这个目录里初始化 Hexo：

```powershell
npx hexo-cli init .
```

初始化完成后，安装主题需要的渲染器：

```powershell
npm uninstall hexo-renderer-marked
npm install hexo-renderer-markdown-it hexo-renderer-sass --save
```

然后把 Aero 主题放到：

```text
themes/aero
```

根目录 `_config.yml` 里把主题改成：

```yml
theme: aero
```

同时把代码高亮的 `hljs` 打开：

```yml
highlight:
  hljs: true
```

## 3. 配置博客基本信息

博客的标题、作者、简介、语言等在根目录的 `_config.yml` 里配置：

```yml
title: 梦不见电子羊的blog
subtitle: live happily.
description: 'Interlinked'
author: 梦不见电子羊
language: zh-CN
timezone: 'Asia/Shanghai'
url: https://sheepblog.xyz
```

其中：

- `title` 是浏览器标题和站点标题
- `subtitle` 显示在顶部横幅
- `description` 显示在头像卡片下方
- `author` 用于版权信息
- `url` 要改成最终的线上域名

## 4. 配置 Aero 主题

Aero 主题自己的配置文件是：

```text
themes/aero/_config.yml
```

头像、社交链接、音乐播放器都在这里改。

例如头像：

```yml
profile:
  enable: true
  author: '梦不见电子羊'
  avatar: '/image/icon/avatar.jpg'
```

社交链接：

```yml
social:
  - name: Github
    link: 'https://github.com/Cantdreamofelectronicsheep'
    icon: /image/icon/github.png
  - name: Bilibili
    link: 'https://space.bilibili.com/399759033'
    icon: /image/icon/bilibili.png
```

注意图片路径要用网页路径，也就是 `/image/...`，不能写成 Windows 的 `\image\...`。

比如：

```md
![丑橘](/image/icon/uglycat.png)
```

不要写成：

```md
![丑橘](\image\icon\uglycat.png)
```

## 5. 添加 About 页面

Aero 主题有 `about` 布局，所以只需要在博客根目录的 `source` 下创建：

```text
source/about/index.md
```

内容示例：

```md
---
title: About
layout: about
date: 2026-05-05 21:30:00
---

![丑橘](/image/icon/uglycat.png)

你好，我是梦不见电子羊。

我在这里写各种我想写的东西，我不知道能坚持多久。

- Email: your-email@example.com
```

导航栏里的 About 会自动跳到：

```text
/about/
```

## 6. 写文章

文章都放在：

```text
source/_posts
```

可以用命令创建：

```powershell
npx hexo new "我的新文章"
```

也可以直接在 VS Code 里新建 `.md` 文件，只要放到 `source/_posts` 里就行。

一篇文章的基本格式是：

```md
---
title: 我的新文章
date: 2026-05-05 23:30:00
tags:
  - 随笔
categories:
  - 生活
---

这里开始写正文。
```

本地预览：

```powershell
npm run build
npm run server
```

然后打开：

```text
http://localhost:4000
```

## 7. 上传到 GitHub

博客源码放在 GitHub 仓库：

```text
https://github.com/Cantdreamofelectronicsheep/blog
```

第一次推送的大致流程：

```powershell
git init
git branch -M main
git remote add origin https://github.com/Cantdreamofelectronicsheep/blog.git
git add -A
git commit -m "Initial Hexo Aero blog"
git push -u origin main
```

如果本地 Git 登录的账号不是仓库所属账号，推送时会出现权限错误。需要重新登录 GitHub 凭据：

```powershell
git credential-manager github logout
git credential-manager github login
```

## 8. 部署到 Cloudflare Pages

博客本体部署在 Cloudflare Pages。

创建 Pages 项目时选择 GitHub 仓库：

```text
Cantdreamofelectronicsheep/blog
```

构建配置：

```text
Production branch: main
Build command: npm run build
Build output directory: public
Root directory: 留空
```

如果 Cloudflare 自动选择了 Yarn，并报类似错误：

```text
YN0028: The lockfile would have been modified by this install
```

说明仓库里同时有 `yarn.lock` 和 `package-lock.json`。我本地一直用 npm，所以删除 `yarn.lock`，保留 `package-lock.json`。

以后只要执行：

```powershell
git push
```

Cloudflare Pages 就会自动重新部署。

## 9. 绑定博客域名

我买了域名：

```text
sheepblog.xyz
```

然后把域名的 DNS 托管到 Cloudflare。

博客域名绑定到 Cloudflare Pages：

```text
https://sheepblog.xyz
https://www.sheepblog.xyz
```

在 Cloudflare Pages 的项目里添加 Custom domains：

```text
sheepblog.xyz
www.sheepblog.xyz
```

等状态变成 `Active`，SSL 显示 enabled，就可以用正式域名访问博客。

## 10. 部署 Waline 评论系统

博客是静态站点，自己不能保存评论，所以需要一个评论服务端。

我选择 Waline。

整体结构是：

```text
Cloudflare Pages 负责博客页面
Vercel 负责 Waline 服务端
Neon 负责保存评论数据
```

先用 Vercel 从 Waline 模板创建项目：

```text
blog-waline
```

然后在 Vercel 的 Storage 里创建 Neon 数据库，并在 Neon 的 SQL Editor 里运行 Waline 的 PostgreSQL 初始化 SQL。

注册管理员地址：

```text
https://你的-waline-服务地址/ui/register
```

第一个注册的用户就是管理员。

管理后台地址：

```text
https://你的-waline-服务地址/ui
```

## 11. 给 Waline 绑定评论域名

一开始 Waline 地址是 Vercel 默认域名：

```text
https://blog-waline-kappa-roan.vercel.app
```

但是有些国内网络访问 `vercel.app` 不稳定，所以我给 Waline 绑定了子域名：

```text
https://comment.sheepblog.xyz
```

Vercel 里添加 Domain：

```text
comment.sheepblog.xyz
```

Cloudflare DNS 里添加 CNAME：

```text
Type: CNAME
Name: comment
Target: cname.vercel-dns.com
Proxy status: DNS only
TTL: Auto
```

注意这里建议用灰色云朵，也就是 `DNS only`。

然后把主题配置里的 Waline 地址改成：

```yml
waline:
  enable: true
  serverURL: 'https://comment.sheepblog.xyz'
```

## 12. 添加留言板和文章评论

我给导航栏新增了一个入口：

```text
留言板
```

页面文件是：

```text
source/guestbook/index.md
```

内容：

```md
---
title: 留言板
layout: guestbook
date: 2026-05-05 23:00:00
---

欢迎来到留言板。

可以在这里随便说点什么，留下路过的痕迹。
```

同时在文章模板 `themes/aero/layout/post.ejs` 底部加入 Waline 评论区。

这样：

- `/guestbook/` 是留言板
- 每篇文章底部都有评论区

## 13. 评论设置

如果不想每条评论都手动审核，可以在 Vercel 的 `blog-waline` 项目里设置环境变量：

```text
COMMENT_AUDIT=false
```

如果想显示评论者的地区、浏览器、系统，可以设置：

```text
DISABLE_REGION=false
DISABLE_USERAGENT=false
```

建议同时设置：

```text
SITE_URL=https://sheepblog.xyz
SERVER_URL=https://comment.sheepblog.xyz
```

改完环境变量后，要在 Vercel 里 Redeploy 一次。

## 14. 音乐播放器优化

一开始我把 mp3 放在博客本地资源里：

```text
themes/aero/source/music
```

这样虽然简单，但音乐文件会跟着博客一起部署到 Cloudflare Pages。手机第一次播放时，如果文件比较大，加载会比较慢。

后来我把音乐压缩到 2.4 MB 左右，并上传到阿里云 OSS。

OSS 上传时选择：

```text
文件 ACL: 公共读
域名: 外网域名
```

然后把主题配置里的音乐地址改成 OSS 链接：

```yml
music:
  enable: true
  autoplay: false
  songs:
    - title: 'ミスティック ブルース'
      artist: '具島直子'
      cover: '/image/icon/ミスティック ブルース.jpg'
      url: 'https://你的-oss-音乐链接.mp3'
```

这样博客仓库不用再带着大 mp3，手机播放速度也会更好。

## 15. 以后更新博客的流程

以后写完文章后，在 VS Code 终端里执行：

```powershell
npm run build
git status
git add -A
git commit -m "Update blog"
git push
```

如果是新增文章，可以先：

```powershell
npx hexo new "文章标题"
```

也可以直接在 VS Code 里新建 `.md` 文件，只要放在：

```text
source/_posts
```

推送之后，Cloudflare Pages 会自动部署。等一会儿，刷新：

```text
https://sheepblog.xyz
```

就能看到更新。

## 结尾

这次搭建过程里，真正重要的不是某一个工具，而是把整条链路跑通：

```text
本地写作 -> GitHub 保存源码 -> Cloudflare Pages 部署博客 -> Waline 保存评论 -> 域名统一访问
```

跑通以后，后面写博客就很简单了。

现在只要打开 VS Code，写一篇 Markdown，然后 `git push`，这个小站就会自动更新。
