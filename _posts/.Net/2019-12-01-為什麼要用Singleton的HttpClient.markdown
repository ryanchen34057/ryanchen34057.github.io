---
layout: post
title: 為什麼要重複使用HttpClient的實體?
categories: [.Net]
description: some word here
keywords: 
---
有研究過`HttpClient`這個類別的工程師們應該都知道，雖然`HttpClient`實作了`IDisposable`這個介面，微軟官方卻建議要重複使用`HttpClient`的實體，但是原因是甚麼呢?如果每次使用完畢就把實體給`Dispose`掉又會有甚麼問題呢?就讓我們一探究竟吧!

## `Dispose` HttpClient實體的問題
**當我們在`Dispose` HttpClient的實體時，HttpClient裡的`HttpClientHandler`也會被`Dispose`掉，這樣會導致連線被中斷。**
而這樣又會造成兩個問題:
* 重新建立連線很慢
* 因為關掉連線需要時間，所以我們不一定有足夠的`socket`來建立新連線

## 重複使用HttpClient實體的問題
我們知道`HttpClient`實體一定要重複使用，不然會有很嚴重的問題，但是重複使用`HttpClient`實體又有甚麼問題呢?
那就是**無法即時反應DNS的變化!**
這樣會造成:
* 有機會讓請求到達不了正確的伺服器
* 在使用Azure Paas等服務時會有問題

為了解決以上的問題，微軟在.NET 2.1時推出了`HttpClientFactory`!

## 介紹HttpClientFactory
`HttpClientFactory`在每次創建新的`HttpClient`實體時並不會`Dispose` `HttpClientHandler`，而是從`HttpMessageHandlerPool`取得實體，然後控制它的生命週期，一個Handler預設是會被抓著2分鐘。所以每個新創建的`HttpClient`實體可以分享前一個連線的Handler及連線。

![](https://i.imgur.com/kA5wApf.png)

這樣很有效地解決了: 
1. 不斷生成socket的問題
2. 因為handler預設在2分鐘後會被`dispose`掉，所以解決了DNS的問題

