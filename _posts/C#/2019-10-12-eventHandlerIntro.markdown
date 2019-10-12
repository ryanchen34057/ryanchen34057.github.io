---
layout: post
title: C# EventHandler 介紹   
categories: [C#]
description: some word here
keywords: C#,EventHandler
---

## 什麼是EventHandler?
*EventHandler*負責用來接收及處理從委派(Delegate)方法傳來的資料。

通常會接受2個參數:
* *Sender*
* *EventArgs*

例子:
```csharp
public void btnSubmit(object sender, EventArgs e)
{
    // 處理按鈕的事件
}
```

>延伸閱讀: []()