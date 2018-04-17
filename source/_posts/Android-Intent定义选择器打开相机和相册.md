---
title: Android Intent定义选择器打开相机和相册
date: 2018-04-17 19:21:56
tags:
- Intent
---

# Android Intent选择打开相机和相册

[CSDN博客对应文章地址](https://blog.csdn.net/lwq573384928/article/details/79979300)
## 需求
上传图片，点击按钮弹出选择相机拍照或者相册选择照片

## 解决方案

Intent.ACTION_CHOOSER

###　打开相机

```
Intent captureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
captureIntent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION |　Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
```
### 打开相册

```
Intent albumIntent = new Intent(Intent.ACTION_PICK, null);
albumIntent.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, "image/*");
```

### 使用Intent.ACTION_CHOOSER将相机和相册合在一起

```
//创建ChooserIntent
Intent intent = new Intent(Intent.ACTION_CHOOSER);
//创建相机Intent
Intent captureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
captureIntent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION |　Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
//将相机Intent以数组形式放入Intent.EXTRA_INITIAL_INTENTS
intent.putExtra(Intent.EXTRA_INITIAL_INTENTS, new Intent{captureIntent});
//创建相册Intent
Intent albumIntent = new Intent(Intent.ACTION_PICK, null);
albumIntent.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, "image/*");
//将相册Intent放入Intent.EXTRA_INTENT
intent.putExtra(Intent.EXTRA_INTENT, albumIntent);       
```

----
Have Fun:)


