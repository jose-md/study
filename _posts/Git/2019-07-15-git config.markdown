---
layout: post
title:  "git config"
date:   2019-07-15 09:20:00 +0800
categories: Git
tags: Git
author: pepe
description: 『 git config 』
---

```
git config user.name     // 查看自己的用户名
git config user.email    // 查看自己的邮箱地址

git config --global user.name "wangp" //修改自己的用户名
git config --global user.email "xxx" //修改自己的邮箱地址
```

```
git remote -v  			// 查看远程仓库地址
git remote set-url origin [url] 	// 修改远程仓库地址

// 先删后加
git remote rm origin 
git remote add origin [url]
```


参考：

[Git查看与修改用户名、邮箱 - _whys - 博客园](https://www.cnblogs.com/wyhlightstar/p/6283517.html)

























