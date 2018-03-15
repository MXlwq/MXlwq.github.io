---
title: Ubuntu ext硬盘权限修改为当前用户
date: 2018-03-15 19:17:48
tags:
- Ubuntu
categories:
- Ubuntu
---
[CSDN博客对应文章地址](http://blog.csdn.net/lwq573384928/article/details/79572347)
## 现象
GUI下某个文件夹内无法创建文件夹、删除文件等操作，只能在终端中用sudo 命令来创建和删除文件。
## 根本原因
当前文件所在的组属于root
## 我的原因
Windows+Ubuntu双系统，后来把Windows系统的分区给格式化了，进入Ubuntu修改为ext4格式，出现上述问题。
## 解决方案

```
whoami #查看当前用户名称
#在出问题的分区中打开终端，可以看到所在位置和分区的名称
sudo chown -R username /media/username/diskname/  #其中的username就是whoami中查出来的，diskname是分区名称
sudo umount /media/username/diskname/
reboot
```
----
That's all, have fun :)


