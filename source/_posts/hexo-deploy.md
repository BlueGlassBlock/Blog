---
title: 给自己开个博客吧！
date: 2022-09-17 10:00:00
tags: [hexo, deploy]
---

# 缘起

最近在鼓捣 `kayaku`，（简单来说，一个挺奇妙的配置库），因为这是我个人第一次独立完成一个项目从顶到底所有的设计工作
（`Ariadne` 不能算，因为那玩意都是从 `Application` 抄过来的），所以干脆就开了个博客。

<!--more-->

# 用什么框架

遵循不造轮子的原则 ~~其实就是不会~~，我选择了和 [`Redlnn`](https://blog.redlnn.top) 一样的 `hexo`，
用观感不错的 [`hexo-theme-fluid`](https://github.com/fluid-dev/hexo-theme-fluid) 作为主题。

# 开造！

`npm install --save hexo-theme-fluid`

`npx hexo new page about`

`npx hexo new post hexo-deploy`.

# 部署

因为我没有钱，所以选择白嫖 `CloudFlare Pages`（不过据说国内访问体验不佳？）

总之我自己能看得到就行了。