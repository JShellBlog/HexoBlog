---
title: hexo_add_search
date: 2019-07-04 09:29:56
tags: hexo
categories: hexo
---

## 1. 前言

当博文慢慢变多的时候，标签和分类已经不能提供太大的作用，无法准确的定位到自己想要看的博客上去，所以添加一个本站内搜索功能是很有必要的。

<!--more-->

## 2. 安装插件

```
npm install hexo-generator-searchdb --save
```

修改blog下的_config.yml文件，进行编辑。

```
search:
    path: search.xml
    field: post
    format: html
    limit: 10000
````


修改主题配置文件blog/themes/next下的_config.yml文件，进行编辑。

```
local_search:
    enable: true
```

## 参看资料
[hexo博客添加搜索功能](https://blog.csdn.net/qq_40265501/article/details/80030627)

[Hexo博客添加站内搜索](https://www.ezlippi.com/blog/2017/02/hexo-search.html)

[Hexo 博客添加本地搜索](https://www.chunqiuyiyu.com/2018/07/hexo-local-search.html)
