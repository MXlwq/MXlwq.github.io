---
title: Android EditText 通过TextWatcher实现自动补全的注意点
date: 2018-03-09 14:50:47
tags:
- Android
- TextWatcher
categories:
- 移动开发
---
# Android EditText 通过TextWatcher实现自动补全的注意点
[CSDN文章对应地址](http://blog.csdn.net/lwq573384928/article/details/79317064)
## 背景
需求想要实现输入框在用户输入了一定文本的情况下 自动填充一个可能用户想要的结果，类似Chrome手机版的搜索框

## 实现

```
private class MyWatcher implements TextWatcher {
        public void afterTextChanged(Editable s) {
            Log.d(TAG, "doAfterTextChanged..");

            if (一些可能防止自动补全的情况) {
                return;
            }

            //为了防止快速输入的情况下重复调用，加一个延时
            mHandler.removeMessages(MSG_AUTO_COMPLETE);
            mHandler.sendEmptyMessageDelayed(MSG_AUTO_COMPLETE, DELAY_AUTO_COMPLETE);
        }

        public void beforeTextChanged(CharSequence s, int start, int count, int after) {

        }

        public void onTextChanged(CharSequence s, int start, int before, int count) {
            //这里可以设置一些标志位，用于防止自动补全，比如这里的识别用户按了删除按键，通过count==0来判断
            mLastUrlEditWasDelete = (count == 0);
        }
    }
```
上面一些可能防止自动补全的情况有很多，一般有：

 1. 用户点了删除键；
 2. 正在处于上一次自动补全的过程中；

## 存在的兼容性问题
功能上线之后，发现国外的用户自动补全后点了删除，会再次自动补全，导致输入框无法删除，这个是很严重的bug，但是在测试和开发的过程中都没有发现，后来发现国外的用户用的是Gboard输入法，默认有一个文本输入建议，而开发和测试都用的国内的输入法，比如讯飞和搜狗等等。这里就出现了输入法的兼容问题，使用不同的输入法，点击删除时回调的TextWatcher的次数不同，在网上找了很多解决方案，最多的就是说让给EditText设置一个输入类型，也就是inputType为textNoSuggestions，还有说用textVisiblePassword的，但是前者textNoSuggestions测试无用，无法控制Gboard不显示输入建议，后者textVisiblePassword会导致只能输入数字和英文，但是海外的用户并不都是输入英文的。所以**这些都不是真正的解决方案** 或者说不适用于目前的情况。
## 解决方案
在是否自动补全的地方下功夫，判断处于输入法的建议补全的时候就不自动补全
```
    private boolean shouldAutoComplete() {
        Editable text = getText();
        return !isHandlingBatchInput()
                && BaseInputConnection.getComposingSpanEnd(text)
                == BaseInputConnection.getComposingSpanStart(text);
    }
```
缺憾，这样的话，只有用户点击了输入法内的搜索建议的词，输入内容才会出现在EditText中，但是此时不会执行自动补全操作，我的解决方案是
使用自定义的InputConnectionWrapper
重写EditText的onCreateInputConnection方法，

```
    @Override
    public InputConnection onCreateInputConnection(EditorInfo outAttrs) {
        mInputConnectionWrapper.setTarget(super.onCreateInputConnection(outAttrs));
        return InputConnectionWrapper;
    }
    InputConnectionWrapper mInputConnectionWrapper = new InputConnectionWrapper(null, true) {
        @Override
        public boolean finishComposingText() {
            autoComplete(mLastSuggestion);
            return super.finishComposingText();
        }
    };
```
在finishComposingText的时候调用一次自己写的autoComplete方法。

-----------------------
Have Fun~
