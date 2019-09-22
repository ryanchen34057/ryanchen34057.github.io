---
layout:     post
title:      "Java 泛型(Generic)裡的extends"
subtitle:   ""
date:       2019-08-31
author:     "Ryan"
header-style: text
tags:
    - Java
---

在看別人的Source Code的時候看到了這個寫法:
```java
public class Util {
    public static <T extends Comparable> void sort(T[] array) {
        sort(array, 0, array.length - 1);
    }
 // 以下省略
 ```
 蝦米阿? 原來`extends`還可以加在泛型裡面喔?

 Google了一下才知道，
 
 ***定義泛型的時候，是可以定義泛型的邊界的。***

 也就是說，

 雖然我可以讓你很方便的放入想放的資料型態，但是最少這個型態必須要滿足我的基本要求!

 像是我要找另一半，但是不是是女生就可以，~~罩杯要C以上~~(威~)

 以上面的例子來說，因為資料要可以比較才可以排序嘛，所以`T`必須是有實作`Comparable`的資料型態才行!

 