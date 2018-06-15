---
title: Android Studio .gitignore修改后不生效的问题
date: 2018-06-01 10:01:18
tags:
---

## 问题
Android Studio .gitignore修改后不生效，没有ignore对应的文件，

## 原因
git没有清理cache重点内容

## 解决方案
```
git rm -r --cached .
git add .
git commit -m 'Your commit-msg'
```


