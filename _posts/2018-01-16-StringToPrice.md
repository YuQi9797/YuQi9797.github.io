---
layout: post  #这个不变
title: "数字字符串转换为货币格式" #标题
date: 2018-01-16 12:53 #时间
description: "将数字字符串转换为货币格式"  #说明
tag: Android #这是分类标签
---

```
package com.qiyu.blogandroiddemo;

import java.text.NumberFormat;
import java.util.Locale;

public class NumberUtils {

  public static String getMoneyType(String string) {
      // 把string类型的货币转换为double类型。  
      Double numDouble = Double.parseDouble(string);
      // 想要转换成指定国家的货币格式
      NumberFormat format = NumberFormat.getCurrencyInstance(Locale.CHINA);
      // 把转换后的货币String类型返回
      String numString = format.format(numDouble);
      return numString;
  }
}

```
我们可以把CHINA替换成其他地方，比如US、UK、ENGLISH、Canada等，就可以转换为对应地方的货币书写格式了。
