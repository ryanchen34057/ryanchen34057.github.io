---
layout: post
title: C# 註冊Event用匿名函數的方法
categories: [C#]
description: some word here
keywords: C#,Event
---

之前介紹過Event以下的用法:
```csharp
var worker = new Worker();
worker.WorkPerformed += new EventHandler<WorkPerformedEventArgs>(Worker_WorkPerformed);

public static void Worker_WorkPerformed(object sender, WorkPerformedEventArgs e)
{
    Console.WriteLine(e.Hours + " " + e.WorkType);
}
```

但是如果不想要特別定義一個函數的話其實可以用匿名函數的方式:
```csharp
var worker = new Worker();
worker.WorkPerformed += delegate(object sender, WorkPerformedEventArgs e)
    {
        Console.WriteLine(e.Hours + " " + e.WorkType);
    }
```

不過這個方法的缺點就是函數無法被重複使用了。