---
title: Android View焦点获取
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
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    View rootview = CurrentActivity.this.getWindow().getDecorView();
                    View focusView = rootview.findFocus();
                    Log.i("TAG", focusView==null?"当前无焦点"+toString());
                }
            }
        });

```
