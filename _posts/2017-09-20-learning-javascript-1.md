---
layout: post
title: js刷新图片源的缓存问题
categories: javascript
description: js刷新图片源的缓存问题
keywords: javascript, js, 图片源缓存
---

# js刷新图片源的缓存问题

## 问题重现

当使用js来刷新一张图片的图片源，如果只是图片变了，而src路径不变（路径和图片名不变，但图片内容变了），则会引发浏览器的缓存问题：

当src路径不变时，浏览器不会重新去请求一次改地址，而是从浏览器自己的缓存里读取

但这样是情有可原的，因为这会大幅提高网页加载速度；那总要解决这个办法吧？看下面

## 解决办法

其实思路很简单，既然是因为路径一样导致的问题，那我每次请求的路径不一样不就行啦？！

最简单有效的办法：在src地址后面再加一个无关的参数（这个参数不能有实际意义），参数的值可以用一个随机数来代替，js里面就有一个生成随机数的方法 ``Math.random()``  ；那解决办法就出来了：

```javascript
path:"./data/imageActionServlet?random="+Math.random()+"&path="+retData.get("path"),
```