---
layout: post
title: C# 問號點(?.)用法介紹   
categories: [C#]
description: some word here
keywords: C#,Programming Language,問號點
---

寫了C#一個多月，今天才看到一個很酷的用法。

以前在寫Java的時候最常碰到的錯誤就是`Null Pointer Exception`，也就是物件在`null`的狀態下我們嘗試去存取裡面的屬性時所產生的錯誤。

所以以前常常會這樣去防:
```java
Object a = GetObject();
if(a != null) {
    a.SomeMethod();
}
```
這樣就可以保證至少程式不會出錯。

不過這樣就會有點冗贅，那號稱糖果包(XD)的C#有什麼厲害的寫法呢!?


塔搭!

```csharp
Object a = GetObject();
a?.SomeMethod(); 
// 如果a是null，整個回傳值會是null，
//如果不是則就會跑SomeMethod()的方法
```