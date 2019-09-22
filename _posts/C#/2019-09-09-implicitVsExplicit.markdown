---
layout:     post
title: C#的隱含型別(Implicit Typed) vs 顯式型別(Explicit Typed)   
categories: [C#]
description: some word here
keywords: C#,Programming Language
---


# 介紹
## 隱含型別(Implicit Typed)
隱含型別使用`var`來宣告，在編譯時才會被指派明確的型別。
```csharp
var grade = 90;
var StreamReader = new StreamReader();
```

## 顯式型別(Explicit Typed)
顯式型別則是在程式碼撰寫時就明確指派型別，範例如下:
```csharp
int i = 1;
int n = 2;
```

## 差別
簡單來說，**根本沒差!XD**

基本上編譯過後兩者的效能會是一模一樣的，所以在效能這點上是不用擔心`var`會比較耗效能。

## 可讀性(Readibility)
效能沒差的話差別就是可讀性囉!

如果想要一眼看出變數是甚麼型別的話用顯式型別式會比較清楚，但是`var`的話就是整體會比較短，就看公司內部`Coding Style`如何規範就用哪個囉!(像我們公司就是要用顯式)

`var`還有一個優點就是假如方法內的區域變數要更改型別，就只要更改參數內的型別就好，假設有個方法這樣寫:

```csharp
public IEnumerable GetStockOverPrice(IEnumerable<Stock> stocks, int priceOver)
{
    IEnumerable<Stock> result = new IEnumerable<Stock>();
    for(int i = 0;i < stocks.Count; i++)
    {
        if(stocks[i] >= priceOver)
        {
            result.Add(stocks[i]);
        }
    }
    return result;
}
```

假設類似的方法在公司的專案裡有1000個，那如果今天要把`IEnumerable`改成`List`的時候就要把區域變數用到`IEnumerable`改成`List`了，光用想的就累死人哈!

用`var`就可以省去修改的時間囉!