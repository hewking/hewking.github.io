---
layout: post
title:  "EditText通过设置TextWatcher，禁止不符合要求字符输入"
crawlertitle: "EditText通过设置TextWatcher"
summary: "EditText通过设置TextWatcher，禁止不符合要求字符输入"
date:   2016-01-20
categories: posts
tags: 'AndroidSource'
author: hewking
---


>因为业务需要，使用EditText，只需要字母数字下划线输入。

EditText 有很多属性可以设置，与需求相关的是inputType.
例如：android:inputType="textGapCharacter"。在这里采取另外的方式实现，EditText 继承 TextView 所以可以设置TextWatcher,能够对各种输入做操作。包括禁止，改变颜色，大小等。

###先贴代码：
```
public class UserNameTextWatcher implements TextWatcher {
    private static final String PASSWORD_REGEX = "[a-zA-Z0-9_]*";

    private boolean mIsMatch;
    private CharSequence mResult;
    private int mSelectionStart;
    private int mSelectionEnd;
    private EditText mPswEditText;


    public UserNameTextWatcher(EditText editText) {
        mPswEditText = editText;
    }

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        mSelectionStart = mPswEditText.getSelectionStart();
    }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        CharSequence charSequence = "";
        if ((mSelectionStart + count) <= s.length()) {
            charSequence = s.subSequence(mSelectionStart, mSelectionStart + count);
        }
        mIsMatch = pswFilter(charSequence);
        if (!mIsMatch) {
            String temp = s.toString();
            mResult = temp.replace(charSequence, "");
            mSelectionEnd = start;
        }
    }

    @Override
    public void afterTextChanged(Editable s) {
        if (!mIsMatch) {
            mPswEditText.setText(mResult);
            mPswEditText.setSelection(mSelectionEnd);
        }
    }

    private boolean pswFilter(CharSequence s) {
        if (TextUtils.isEmpty(s)) {
            return true;
        }
        Pattern pattern = Pattern.compile(PASSWORD_REGEX);
        Matcher matcher = pattern.matcher(s);
        if (matcher.matches()) {
            return true;
        }
        return false;
    }
```

###这里首先详细描述下，TextWatcher 接口的三个方法。
```
public void onTextChanged(CharSequence s, int start, int before, int count)
```
该方法在文本改变之前调用，传入了四个参数：CharSequence s：文本改变之前的内容
int start：文本开始改变时的起点位置，从0开始计算
int count：要被改变的文本字数，即将要被替代的选中文本字数
int after：改变后添加的文本字数，即替代选中文本后的文本字数
该方法调用是在文本没有被改变，但将要被改变的时候调用，把四个参数组成一句话就是： 
在当前文本s中，从start位置开始之后的count个字符（即将）要被after个字符替换掉

```
public void onTextChanged(CharSequence s, int start, int before, int count)
```
该方法是在当文本改变时被调用，同样传入了四个参数：
CharSequence s：文本改变之后的内容
int start：文本开始改变时的起点位置，从0开始计算
int before：要被改变的文本字数，即已经被替代的选中文本字数
int count：改变后添加的文本字数，即替代选中文本后的文本字数
该方法调用是在文本被改变时，改变的结果已经可以显示时调用，把四个参数组成一句话就是： 
在当前文本s中，从start位置开始之后的before个字符（已经）被count个字符替换掉了

```
public void afterTextChanged(Editable s)
```
该方法是在文本改变结束后调用，传入了一个参数：
Editable s：改变后的最终文本
该方法是在执行完beforeTextChanged、onTextChanged两个方法后才会被调用，此时的文本s为最终显示给用户看到的文本。我们可以再对该文本进行下一步处理，比如把文本s显示在UI界面上

### 实例

```
mEditUsername.addTextChangedListener(new UserNameTextWatcher(mEditUsername));
```