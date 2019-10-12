---
layout: post
title: C# 事件(Event) 介紹   
categories: [C#]
description: some word here
keywords: C#,Event,C#事件
---

## 什麼是Event?
簡單來說，*Event*其實就是通知(Notifications)，如果某個方法(例如按按鈕)跟某個事件(另外一個方法)綁在一起，那做了這件事情之後就會觸發事件，也就是做了某個方法會觸發另一個方法。

## Event的特點
* 綁定Event的物件不需要清楚知道是哪個物件在處理Event的
* Event透過*EventArgs*來傳遞資料

## Event實作方式
如果要自己定義Event，要使用`event`關鍵字:
```csharp
// Delegate
public delegate void WorkPerformedHandler(int hours, WorkType workType);

public enum WorkType
{
    GoToMeetings,
    Golf,
    GenerateReports
}

public event WorkPerformedHandler WorkPerformed;
// WorkPerformedHandler -> 是Delegate!!
```

**呼叫Event**

當作方法呼叫即可:
```csharp
if(WorkPerformed != null)
{
    WorkPerformed(8, WorkType.GenerateReports);
}
```

因為`WorkPerformed`其實是`delegate`，所以我們也可以明確轉型成`delegate`來使用:
```csharp
WorkPerformedHandler del = WorkPerformed as WorkPerformedHandler;
if(del != null)
{
    del(8, WorkType.GenerateReports);
}
```

以下是完整的範例:

```csharp
public delegate void WorkPerformedHandler(int hours, WorkType workType);
public class Worker
{
    public event WorkPerformedHandler WorkPerformed; // Event的定義
    public virtual void DoWork(int hours, WorkType workType)
    {
        // 做一些事,然後通知用戶方法已被呼叫
        OnWorkPerformed(hours, workType);
    }

    protected virtual void OnWorkPerformed(int hour, WorkType workType)
    {
       if(WorkPerformed != null)
       {
           WorkPerformed(hours, workType);
       }
    }

    // ------ 或是 ------

    protected virtual void OnWorkPerformed(int hour, WorkType workType)
    {
        WorkPerformedHandler del = WorkPerformed as WorkPerformedHandler;
        if(del != null)
        {
            del(hours, workType); // 呼叫Event
        }
    }
}
```