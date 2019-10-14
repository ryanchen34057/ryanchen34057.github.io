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

以下是範例:

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

## 使用EventArgs
一般比較建議的做法是讓EventHandler有以下的格式:
```csharp
public delegate void WorkPerformedHandler(object sender, EventArgs e);
```

`sender`代表啟動Event的方法，而`EventArgs`則是Event自帶的資料。

如果要自己定義這個`EventArgs`，我們可以自己寫一個類別，然後來繼承至`System.EventArgs`:

```csharp
public class WorkPerformedEventArgs : System.EventArgs
{
    public int Hours {get; set; }
    public WorkType WorkType {get; set; }
}
```

## 使用EventHandler來註冊及觸發Event
如果沒看過*EventHandler*請參照: [C# EventHandler 介紹](https://ryanchen34057.github.io/2019/10/12/eventHandlerIntro/)

首先我們可以將`delegate`的`WorkPerformedHandler`改寫成以下的程式碼:
```csharp
public event EventHandler<WorkPerformedEventArgs> WorkPerformed;
```

我們可以透過以下的方式來註冊`EventHandler`到`Worker`類別:
```csharp
var worker = new Worker();
worker.WorkPerformed += 
    new EventHandler<WorkPerformedEventArgs>(worker_WorkPerformed);

void worker_WorkPerformed(object sender, WorkPerformedEventArgs e)
{
    Console.WriteLine(e.Hours.ToString());
}
```

## 完整範例
接下來來改寫一下上面的`Worker`類別，然後做一個簡單的Demo:
```csharp
//public delegate void WorkPerformedHandler(int hours, WorkType workType);
public class Worker
{
    public event EventHandler<WorkPerformedEventArgs> WorkPerformed; // Event的定義
    public event EventHandler WorkCompleted; 

    public virtual void DoWork(int hours, WorkType workType)
    {
        for(int i = 0;i < hours; i++)
        {
            // 每小時通知一次
            OnWorkPerformed(i + 1, workType);
        }
        // 結束時再通知一次
        OnWorkCompleted();
        
    }

    protected virtual void OnWorkPerformed(int hours, WorkType workType)
    {
       var del = WorkPerformed as EventHandler<WorkPerformedEventArgs>;
       if(del != null)
       {
           del(this, new WorkPerformedEventArgs(hours, workType))
       }
    }

    protected virtual void OnWorkCompleted()
    {
        var del = WorkCompleted as EventHandler;
        if(del != null)
        {
            del(this, EventArgs.Empty); // 如果不打算帶資料可以使用EventArgs.Empty
        }
    }
}
```

然後寫一個簡單的Console App來測試一下結果:
```csharp
public class Program
{
    public static void Main()
	{
		var worker = new Worker();
        worker.WorkPerformed += new EventHandler<WorkPerformedEventArgs>(Worker_WorkPerformed);
        worker.WorkCompleted += new EventHandler(Worker_WorkCompleted);
        worker.DoWork(8, WorkType.GenerateReports);

        Console.Read();

	}

    public static void Worker_WorkPerformed(object sender, WorkPerformedEventArgs e)
    {
        Console.WriteLine(e.Hours + " " + e.WorkType);
    }

    public static void Worker_WorkCompleted(object sender, EventArgs e)
    {
        Console.WriteLine("Work Completed!");
    }
}
```

測試結果如下:
```
1 GenerateReports
2 GenerateReports
3 GenerateReports
4 GenerateReports
5 GenerateReports
6 GenerateReports
7 GenerateReports
8 GenerateReports
Work Completed!
```