---
layout: post
title: C# 利用延遲實體化(Lazy Initialization)來節省資源及提升效能   
categories: [C#]
description: some word here
keywords: C#,Lazy
---

## 介紹
延遲實體化(Lazy Initialization)是一種技巧，可以用來拖延實體化的時間點，到了需要該資源的時候才會去實體化它。

我們可以用延遲實體化的技巧來避免一些不必要的計算，這麼做可以提升**程式效能**及**記憶體消耗**。

我們可以用顧客`Customer`及訂單`Order`的例子來說明這個概念。

假設`Customer`類別裡有屬性`Orders`，存放著大量的訂單資料，這些資料有可能是需要從資料庫提取出來的。

這樣一來，在真正需要訂單資料以前，我們其實不需要先把所有`Orders`的資料都load進來。

`Lazy`讓我們可以在需要資料時才去做實體化的動作。

## 實作Lazy<T>類別
在使用`Lazy<T>`類別時，我們需要指定物件的型別，然後等到我們存取`Lazy<T>.Value`屬性之後，該物件才會被實體化出來。

```csharp
Lazy<IEnumerable<Order>> LazyOrders = new Lazy<IEnumerable<Order>>();
IEnumerable<Order> result = lazyOrders.Value; // <- 這時候才會被實體化!
```

我們再來看另外一個例子:

假設有`Arthor`跟`Book`類別:

```csharp
public class Author
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
    public List<Book> Books { get; set; }
}
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public DateTime PublicationDate { get; set; }
}
```

`Books`可以通過某些資料庫提取的方式取得。

但是假設在使用者的介面上顯示時只會秀出作者的基本資料，不會去顯示他的著作，那我們其實就不需要事先將`Books`的資料都從資料庫取回來對吧? 這時候就可以Being Lazy一下，

等到真的需要這些`Books`的資料再做`GetBookDetailsForAuthor`裡的動作。

```csharp
public class Author
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
    public Lazy<IList<Book>> Books => new Lazy<IList<Book>>(() => GetBookDetailsForAuthor(this.Id));
    private IList<Book> GetBookDetailsForAuthor(int Id)
    {
        // 取回Book的資料
    }
}
```