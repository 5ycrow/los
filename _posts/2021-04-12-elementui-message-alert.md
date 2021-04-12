---
layout: post
title: Vue中添加$alert的messageBox消息弹出框，进行换行、空格等html渲染
date: 2021-04-12
Author: Los
tags: [vue,elementui]
comments: false
---



**出现情况：**
在Vue中 $alert的messageBox 无法直接通过正则表达式进行来进行换行和缩进
**解决方式：**
dangerouslyUseHTMLString 是否将 message 属性作为 HTML 片段处理 boolean — false

```js
this.$alert('我需要</br>换行',{dangerouslyUseHTMLString:true})
```