---
title: Ubuntu状态栏和侧边栏消失解决方案——删除iBus引起
date: 2018-03-09 14:42:52
tags:
- Ubuntu
categories:
- Ubuntu
---

# Ubuntu删除iBus后状态栏和侧边栏消失解决方案
[CSDN文章对应地址](http://blog.csdn.net/lwq573384928/article/details/79484342)
## 起因
看到网上的优化教程说用了fctix就不用iBus了，于是就卸载了iBus，没想要的是因为有很多其他的软件会有依赖，比如和界面现实有关系的Unity。

## 解决方案

终端下输入

```
dconf reset -f /org/compiz #重置compiz
sudo apt install unity #如果unity被卸载了，重装unity
setsid unity #重启uni
```

