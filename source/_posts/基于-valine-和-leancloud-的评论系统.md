---
title: 基于 valine 和 leancloud 的评论系统
author: changqing
categories:
  - 2024
date: 2024-02-08 14:36:48
tags:
  - 工具
  - 教程
---


# 评论系统
基于 hexo next 主题搭建博客后，测试了几种评论系统，像畅言、Gitalk 等都需要登陆后才能评论，实际用起来比较繁琐，而且评论的人基于安全的考虑也不会授权第三方评论系统获得自己的账号授权。

试想，一个偶然发现的网站，一个没怎么听到过的系统，怎么会允许获取自己的账号授权呢。

<!-- more -->

# leancloud & valine
leancloud 是一个云计算平台，可以在上边创建应用，然后通过 http 或者其他方式调用平台对外提供的接口。最重要的是，平台提供的接口调用和存储功能都免费！！！

而 valine 就是基于这一特性，在 hexo 主题中嵌入评论插件，用户提交评论时会发数据发送到 leancloud 对外开放的接口，从而实现保存评论信息。

而且，该插件不需要登录。这样评论本身就没有了限制，大家可以畅所欲言，比畅言更畅言。但这种方式对博主本身的安全性有一定的影响，比如恶意评论导致 leancloud 的接口频繁调用。

# 接入方式
下面是以 NexT(7.8.0) 为例进行接入说明。

## 注册 leancloud
登录 [leancloud](https://console.leancloud.app/) 注册账号。
![账号注册](/imgs/2024/0208_leancloud_register.png)

注意一定要用国际版，国内版存在个人信息验证问题！！！


注册成功并登录后，需要邮箱验证。

验证成功后就可以创建自己的应用了，在首页点击：Create App
![创建 app](/imgs/2024/0208_leancloud_create_app.png)

选择开发者模式
![选择开发者模式](/imgs/2024/0208_leancloud_create_app_dev.png)

创建完成过后点【设置】查看 app id 等信息。
![获取 AppId 等信息](/imgs/2024/0208_leancloud_app_setting.png)

图上标记的信息需要在下面 NexT 主题中配置。

## 配置 NexT
在 _config.yml 中找到评论相关配置，设置评论方式为 valine，然后设置 app id 等信息
``` yaml
comments:
  # Choose a comment system to be displayed by default.
  # Available values: changyan | disqus | disqusjs | gitalk | livere | valine
  active: valine

valine:
  enable: true
  appid: xxxx # Your leancloud application appid
  appkey: xxxx # Your leancloud application appkey
  serverURLs: https://xxxx # 会往这个地址中发送数据
```

## 效果
![评论](/imgs/2024/0208_valine_comment.png)
