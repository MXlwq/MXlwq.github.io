---
title: Android View焦点获取
date: 2018-03-09 14:42:52
tags:
- Android
- View
categories:
- 移动开发
---
# 需求来源

测试给提了一个bug，发现是焦点转移导致的，想看看是从哪到哪儿转移了。

# 方案

为了定时输出当前获取焦点的控件信息，可以新开一个线程，每n秒调用一下rootview的findFocus

具体如下
```
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    View rootview = BrowserActivity.this.getWindow().getDecorView();
                    View focusView = rootview.findFocus();
                    Log.i("LWQ", focusView == null ? "当前无焦点" : focusView.toString());
                }
            }
        }).start();

```
