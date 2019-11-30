---
layout: post
title: .NET Core - 用HttpClient呼叫API
categories: [.Net Core]]
description: some word here
keywords: 
---

# 處理CRUD操作
先`new`一個`HttpClient`的實體出來，並設定`BaseAddress`及`Timeout`。這邊建議`new`一個`static`實體出來，因為我們只會需要一個實體。

```csharp
private static HttpClient HttpClient = new HttpClient();

public SomeClass()
{
    HttpClient.BaseAddress = new Uri("http://localhost:1000");
    HttpClient.Timeout = new TimeSpan(0, 0, 30);
    HttpClient.DefaultRequestHeaderS.Clear(); // 最好先Clear比較保險，以免有其他程式碼已經有設定過了!
    HttpClient.DefaultRequestHeaderS.Accept.Add(
        new MediaTypeWithQualityHeaderValue("application/json"); // 告訴API我們只接受json格式的訊息
    )
}
```

※ 注意: 雖然`HttpClient`有實作`IDispoable`的介面，但是微軟官方非常不介意以下寫法，原因之後會詳述:
```csharp
using(var client = new HttpClient())
{
    // Don't do this!!!
}
```
## GET
```csharp
class Product 
{
    public string Id {get; set;}
    public int Price {get; set;}
}
```
這邊我們假設有一個有一道API可以讓我們把所有商品的資訊取回。
等到取得回傳的字串後，再把字串轉為`Product`的物件。

我們有兩種做法: 1. 直接只用`HttpClient`的`GetAsync` 2. 用`HttpRequestMessage`

做法1:
```csharp
var response = await HttpClient.GetASync("api/products");

response = EnsureSuccessStatusCode(); // 確保回傳成功

var content = await response.Content.ReadAsStringAsync();
var products = JsonConvert.Deserialize<IEnumerable<Product>>(content);
```

做法2:
```csharp
var request = new HttpRequstMessage(HttpMethod.Get, "api/products");
reuqest.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
var response = await HttpCliet.SendAsync(request);

response.EnsureSuccessStatusCode();

var content = await response.Content.ReadAsStringAsync();
var products = JsonConvert.DeserializeObject<List<Product>>(content);
```

可以發現做法1只能使用在`HttpClient`所做的設定，但是做法2可以針對每個請求去調整可接受的格式設定，雖然撰寫上比較麻煩，但是這樣可以比較清楚的知道每一次呼叫所做的設定。

## POST
```csharp
var product = new Product()
{
    Id = "123",
    Price = 100
}

var serializeProduct = JsonConvert.SerializeObject(product);

var request = new HttpRequestMessage(HttpMethod.Post, "api/createProduct");

request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

request.Content = new StringContent(serializeProduct);
request.Content.Headers.ContentType = new MediaTypeHeaderValue("applciation/json");

var response = await HttpClient.SendAsync(request);
response.EnsureSuccessStatusCode();

var content = await response.Content.ReadAsStringAsync();
```
`HttpRequestMessage.Content`的型別是`HttpContent`，我們可以依照使用情境選擇`StringContent`、`ObjectContent`、`ByteArrayContent`、`StreamContent`等子類別使用。

## Update
```csharp
var product = new Product()
{
    Id = "345",
    Price = 200
}

var serializedProductToUpdate = JsonConvert.SerializeObject(product);

var request = new HttpRequestMessage(HttpMethod.Put, "api/updateProduct/123");
request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

request.Content = new StringContent(serializeProduct);
request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");

var response = await HttpClient.SendAsync(request);
response.EnsureSuccessStatusCode();

var content = await response.Content.ReadAsStringAsync();

```

## DELETE
```csharp
var request = new HttpRequestMessage(HttpMethod.Delete, 
    "api/deleteProduct/123");
request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

var response = await HttpClient.SendAsync(request);
response.EnsureSuccessStatusCode();

var content = await response.Content.ReadAsStringAsync();
```

