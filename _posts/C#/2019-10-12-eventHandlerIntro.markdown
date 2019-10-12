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
// EventHandler
public void btnSubmit(object sender, EventArgs e)
{
    // 處理按鈕的事件
}
```

## EventArgs?
*EventArgs*是一種比較常見且建議的委派傳遞資料的方式，開發者如果要傳送自訂義的資料型態也可以繼承`EventArgs`的類別。

*EventArgs*:
```csharp
public class WorkPerformedEventArgs : System.EventArgs
{
    public int Hours {get; set; }
    public WorkType WorkType {get; set; }
}
```

*Delegate Handler*:
```csharp
public delegate void WorkPerformedHandler(object sender, WorkPerformedEventArgs e);
```

但是`EventHandler`類別提供一個更簡潔的方式來寫上面的範例:
```csharp
public event EventHandler<WorkPerformedEventArgs> WorkPerformed;
```


>延伸閱讀: 

[C# 事件(Event) 介紹](https://ryanchen34057.github.io/2019/10/12/eventIntro/)<br>
        
[C# 委派(Delegate) 介紹](https://ryanchen34057.github.io/2019/10/12/delegateIntro/)