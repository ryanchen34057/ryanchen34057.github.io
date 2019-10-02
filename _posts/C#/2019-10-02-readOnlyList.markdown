---
layout: post
title: C# IReadOnlyList用法   
categories: [C#]
description: some word here
keywords: C#,IReadOnlyList
---

假設我們有個簡易的帳戶類別 `Account`:
```csharp
using System.Collections.Generic;

namespace DemoLibrary
{
    public class Account
    {
        public string AccountName { get; private set; }
        public decimal Balance { get; private set; }
        public List<string> Transactions { get; private set;}

        public Account() 
        {
            Transaction = new List<string>();
        }
    }
}
```

屬性有:
* `AccountName` - 帳戶名稱
* `Balance` - 帳戶餘額
* `Transactions` - 交易紀錄

屬性都設為`private set`了所以可以確保外界沒辦法更改我們的屬性。

**是嗎?**

![](https://i.imgur.com/O3ipxkA.png)

因為`List`的變數裡的值只是一串地址(`List`的頭)，所以`private set`只是讓你不能夠修改那串地址，但是她並不會阻止你去修改裡面的值，創建實體後會發現`Transactions`還是可以`Add`、`Insert`甚至是`Remove`。

所以就算設了`private set`，外界只要能`get`到交易紀錄的屬性都能輕易地修改裡面的值。

那要怎麼辦呢? 把`Transaction`改成`IReadOnlyList`就可以囉!

範例:
```csharp
using System.Collections.Generic;

namespace DemoLibrary
{
    public class Account
    {
        public string AccountName { get; private set; }
        public decimal Balance { get; private set; }
        private List<string> _transactions;

        public IReadOnlyList<string> Transactions
        {
            get { return _transactions.AsReadOnly(); }
        }

        public Account() 
        {
            _transactions = new List<string>();
        }
    }
}
```

修改好後就會發現沒辦法去修改`List`裡面的值囉!

![](https://i.imgur.com/D4Wsj4e.png)