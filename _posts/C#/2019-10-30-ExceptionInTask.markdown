---
layout:     post
title: 處理在Task裡的例外(Exception)   
categories: [C#]
description: some word here
keywords: C#,Programming Language
---

今天在寫WCF的服務的時候，發現以下的程式碼就算出現丟了Exception也不會去寫Log:
```csharp
public Task<bool> CheckAuthAsync(string id, int appId)
{
    try
    {
        return Body.CheckAuthAsync(id, appId);
    }
    catch (Exception ex)
    {
        Logger.Log(ExceptionLog.Create(ex));
        throw;
    }
}
```

想說只要在`try`底下的程式碼出現的錯誤應該都可以正確抓到，然後寫例外的訊息到Log裡...

但是詢問公司同事之後才知道，

**`Task`裡的`Exception`如果不`await`的話是抓不到的阿!!**

**`Task`裡的`Exception`如果不`await`的話是抓不到的阿!!**

**`Task`裡的`Exception`如果不`await`的話是抓不到的阿!!**

改成下面這樣就可以順利Log了!
```csharp
public async Task<bool> CheckAuthAsync(string id, int appId)
{
    try
    {
        return await Body.CheckAuthAsync(id, appId);
    }
    catch (Exception ex)
    {
        Logger.Log(ExceptionLog.Create(ex));
        throw;
    }
}
```

下次決不會踩同樣的坑了.....