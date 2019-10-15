---
layout: post
title: 『System.ArgumentException 目的陣列不夠長。請檢查 destIndex 與長度，以及陣列的下限 』List執行緒問題解決方法
categories: [C#]
description: some word here
keywords: C#
---

## 錯誤解說
今天公司的程式出現了一個Bug，錯誤如下:
```
System.ArgumentException: 目的陣列不夠長。請檢查 destIndex 與長度，以及陣列的下限。
```
後來發現是下面這段Code的問題:
```csharp
dataGridView.DataSource = new List<Log>(LogInfos);
```

因為`LogInfos`這個`List`會被其他的執行緒改動，所以在把`LogInfos`丟進`new List<>()`裡面觸發`Array.Copy`到新的`List`的同時被其他執行緒更動到了，所以才發生錯誤!

## 解決方法
後來查資料後找到一個不用在程式碼加`Lock`的方法。


把`LogInfos`再用一個自定義的物件包起來，如下:
```csharp
public class Container<T>
{
    private ImmutableList<T> _items = ImmutableList<T>.Empty;
    private IReadOnlyList<T> Items { get { return _items; } }
    public void Add(T item) { _items = _items.Add(item); }
    public List<T> ToList() 
    {
        return Items.ToList();
    }
}
```
裡面的資料結構使用`ImmutableList`，然後要存取資料的時候用`IReadOnlyList`，把`Add`跟`Get`分開，這樣執行緒就不會互相衝突了!

但是這麼做會有幾個缺點:
1. 因為`ImmutableList`內部是用`BinaryTree`去實現的，所以`Add`的時間複雜度不是`O(1)`而是`O(log N)`!所以如果會頻繁的`Add`的話就不建議用。
2. 有可能會發生*Stale Read*，意思就是存取資料的時候，資料並沒有反應到最近一次的更新。這個狀況會發生在`ToList`的同時有人`Add`，這樣雖然不會有`Exception`，但是讀到的資料不會包括同時新增的那一筆。
所以這個方法要能容許這個狀況的發生。
