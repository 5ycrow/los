---
layout: post
title: github文件夹有白色箭头并且不能打开的解决办法
date: 2021-04-13
Author: Los
tags: [git]
comments: false
---

最近在写一个架构系统的demo，因为里面有几个子系统是clone别人的项目，导致github这个文件夹上显示白色箭头并且不能打开。

原来是因为这个文件夹里面有.git隐藏文件，github就将他视为一个子系统模块了。

解决办法：

1、删除文件夹里面的.git文件夹

2、执行git rm --cached [文件夹名]

3、执行git add [文件夹名]

4、执行git commit -m "msg"

5、执行git push origin [branch_name] 
