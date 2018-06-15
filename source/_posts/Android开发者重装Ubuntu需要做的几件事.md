---
title: Android开发者重装Ubuntu需要做的几件事
date: 2018-05-30 10:01:33
tags:
- Ubuntu
categories:
- Ubuntu
---

# 提示
文中所有蓝色文字都是可以点击跳转的对应详细地址
#  更新

```
sudo apt-get update
```
# 安装系统工具
## Vim

```
#安装
sudo apt install vim
#配置
vi ~/.vimrc
```

## [Git](https://blog.csdn.net/yhl_leo/article/details/50760140)

```
sudo apt-get install git
```

## [Python](https://blog.csdn.net/huanhuanq1209/article/details/72673236)

```
sudo apt-get install python
sudo apt-get install python3
```
## [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

```
解压即可
```

# 安装常用软件
## Chrome

```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add
sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
sudo apt-get update
sudo apt-get install google-chrome-stable
```

## 搜狗输入法

```
# 安装fcitx键盘输入法系统
sudo add-apt-repository ppa:fcitx-team/nightly
sudo apt-get update
sudo apt-get install fcitx
sudo apt-get install fcitx-config-gtk
sudo apt-get install fcitx-table-all
sudo apt-get install im-switch  # 输入Y
# 下载搜狗输入法
https://pinyin.sogou.com/linux/?r=pinyin

sudo apt-get install -f
sudo dpkg -i sogoupinyin_***.deb
# 系统默认键盘输入法系统从ibus修改为fcitx
# 在安装的Fcitx配置中(如果没有)添加搜狗输入法
# Logout当前用户
```
## 截图工具[Shutter](https://blog.csdn.net/hanshileiai/article/details/46843713)

```
# 安装
sudo add-apt-repository ppa:shutter/ppa
sudo apt-get update
sudo apt-get install shutter
# 设置快捷键
系统设置的键盘设置(详见Shutter对应地址)
```
## indicator-sysmonitor

```
sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor  
sudo apt-get update  
sudo apt-get install indicator-sysmonitor 
```

# 安装IDE
## [Android Studio](https://developer.android.com/studio/)

添加Android Studio图标到左侧的工具栏
在Android Studio时Configure中选择Create Desktop Entry
![这里写图片描述](https://img-blog.csdn.net/2018053018572183?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x3cTU3MzM4NDkyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
或者在Android Studio启动后在 Help中的Find Action搜索Create Desktop Entry
![这里写图片描述](https://img-blog.csdn.net/2018053018573037?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x3cTU3MzM4NDkyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


#  配置环境变量
## 修改文件

```
sudo gedit .bashrc
```

## JAVA

```
JAVA_HOME=/usr/java/jdk1.8.0_101 #Your Java path
JRE_HOME=$JAVA_HOME/jre
JAVA_BIN=$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```
## Android 

```
export ANDROID_HOME=/home/android/Sdk
export PATH=$PATH:$ANDROID_HOME/tools/:$ANDROID_HOME/platform-tools/
```

## 添加快捷命令

```
alias upload="repo upload ."
alias pull="git pull --rebase"
alias ba="git checkout alpha"
alias bd="git checkout dev"
alias add="git add ."
alias cm="git commit -s"
alias in="adb install -r"
alias ind="adb install -r /home/app/Demo/build/outputs/apk/debug.apk"
alias inr="adb install -r /home/app/Demo/build/outputs/apk/release.apk"
alias asd="./gradlew assembledebug"
alias asr="./gradlew assemblerelease"
```
