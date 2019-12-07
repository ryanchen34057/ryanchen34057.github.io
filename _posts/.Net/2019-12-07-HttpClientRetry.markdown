---
layout: post
title: 分享HttpClient Retry機制的寫法
categories: [.Net]
description: some word here
keywords: 
---
廢話不多說，直接上碼:
```csharp
 static async Task<string> DownloadStringWIthTimeout(string uri)
{
    using(var client = new HttpClient) // 只是範例，我知道HttpClient應該要重複使用!
    {
        var downloadTask = client.GetStringAsync(uri); // 從網路上下載某些東西
        var timeoutTask = Task.Delay(TimeSpan.FromSeconds(3)); // 設定Timeout時間

        var completeTask = await Task.WhenAny(downloadTask, timeoutTask);

        if(completeTask == timeoutTask)
        {
            return null;
        }
        return await downloadTask;
    }
}
```

`Task.WhenAny(Task task1, Task task2)`的作用是當參數中的任何`Task`結束了就會回傳那個先結束的`Task`實體，我們可以利用這個方法來判斷先結束的`Task`是`downloadTask`還是`timeoutTask`。