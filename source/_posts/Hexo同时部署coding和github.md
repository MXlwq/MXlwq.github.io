---
title: Hexo同时部署coding和github
date: 2018-02-30 10:01:33
tags:
- Hexo
categories:
- Hexo
---

# 阿里云解析配置

1. ping ×××.coding.me和×××.github.io获取到国内coding服务器ip地址和国外github服务器地址
2. 将这两个ip添加为A记录，主机记录为www，记录值填写ping出来的地址，将coding的ip设置为默认的，将github的ip设置为【世界】
3. 添加两个CNAME记录类型，主机类型为@，解析路线×××.coding.me

【截图待补充】
