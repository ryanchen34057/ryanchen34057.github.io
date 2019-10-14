---
layout: post
title: C# 委派(Delegate) 介紹   
categories: [C#]
description: some word here
keywords: C#,Delegate,C#委派
---

## 什麼是委派(Delegate)?
*Delegate*其實就是**指向函數的指標(Pointer)**。

使用面向很廣，但其中一個特別重要的是**用來當作*Event*及*EventHandler*之間的傳遞橋梁(Pipeline)**。

>延伸閱讀: [C# 事件(Event) 介紹](https://ryanchen34057.github.io/2019/10/12/eventIntro/)

聽到指向函數的指標可能還是有點模糊，這邊用按鈕的例子來解釋。

假設一個按鈕`Button`綁定了`ClickEvent`，當`ClickEvent`被觸發時，會呼叫`SomeMethod`。

阿要怎麼呼叫呢? 就是要通過Delegate拉~

## Delegate實作方式
如果要自己定義委派方法，要使用`delegate`關鍵字:
```csharp
// Delegate
public delegate void WorkPerformedHandler(int hours, WorkType workType);

public enum WorkType
{
    GoToMeetings,
    Golf,
    GenerateReports
}
```

剛剛有提到Delegate其實就是一個傳遞的橋樑，所以上面的例子只是把`hour`及`workType`這兩個資料透過`WorkPerformedHandler`這個方法傳遞給接收者，接收者必須要有相同數量及型別的參數(變數名稱不受限制):
```csharp
// Handler
public void WorkPerformed1(int workHours, WorkType wtype) 
{
    Console.WriteLine("WorkPerformed1 called");
}
```

那要怎麼把Delegate跟Handler綁在一起呢?只要把Delegate的方法傳入Handler就好!
```csharp
// Delegate Instance
WorkPerformedHandler del1 = new WorkPerformedHandler(WorkPerformed1);
```

而要呼叫`del1`的委派方法時，只要傳入跟`WorkPerformedHandler`一樣的參數就好:
```csharp
del1(5, WorkType.Golf):
```

## 加入更多方法到調用列表(Invocation List)
委派只能綁定一種方法嗎? 其實他是可以綁定複數個方法的!

因為`Delegate`類別繼承了`MulticastDelegate`這個類別，內部實現了`InvocationList`來達到*多點傳送*。

也就是說只要有一個新的委派方法傳進來就會被加入`InvocationList`裡，而每次Handler被呼叫時會自動呼叫`InvocationList`裡所有的委派方法。

實作方式:

```csharp
WorkPerformedHandler del1 = new WorkPerformedHandler(WorkPerformed1);
WorkPerformedHandler del2 = new WorkPerformedHandler(WorkPerformed2);

// 加入至InvocationList
del1 += del2;

// del2也會被呼叫到
del1(5, WorkType.GoToMeetings);
```

另外我們也可以把Handler傳入某個方法的參數中，藉由呼叫該方法來呼叫Handler所綁定的委派方法，例如:
```csharp
static void DoWork(WorkPerformedHandler del) 
{
    del(5, WorkType.GoToMeetings);
}
```



>本文是我在Pluralsight上的課[C# Events, Delegates and Lambdas](https://app.pluralsight.com/library/courses/csharp-events-delegates/table-of-contents)時所作的筆記，有興趣可以看看