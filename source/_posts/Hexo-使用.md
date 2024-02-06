---
title: Hexo 使用
date: 2024-02-06 17:23:07
author: changqing
tags: 
- 教程
categories:
- 2024
---


# 使用 Hexo 搭建网站
> 这里只是记录一些关键点，具体操作步骤参考官网：https://hexo.io/

## 创建新文章

**生成文章**
``` shell
hexo new 'title'
```

**设置目录和标签**
通过 tags、categories 设置，如下
``` html
tags: 
- 教程
categories:
- 2024
```

**上传图片**
把图片放到 /source/imgs/ 下，通过如下方式引用：
```
![window 桌面图片](/imgs/2024/20240205.jpg)  // markdown 语法
```
![window 桌面图片](/imgs/2024/20240205.jpg)

**编译成静态 HTML 文件**
``` shell
hexo g
```

**本地启动服务，预览结果**
``` shell
hexo s
```

# 部署到 Github
需要在 _config.yml 中配置 github 地址和推送分支信息
然后依次执行以下命令：
``` shell
hexo clean
hexo g
hexo d
```