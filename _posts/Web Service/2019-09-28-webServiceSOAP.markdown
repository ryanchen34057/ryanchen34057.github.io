---
layout: post
title: 淺談Web Service(三) - SOAP 簡單物件存取協議
categories: [Web Service]
description: RPC, SOAP, REST
keywords: Web Service, SOA, RPC, SOAP, REST
---

上集: [淺談Web Service(二) - RPC 遠程方法調用](https://ryanchen34057.github.io/2019/09/28/webServiceCommonPractice/)

今天來換介紹SOAP ~~肥皂(誤)~~

## SOAP(Simple Object Access Protocol))介紹
上次介紹了RPC，RPC的概念為調用伺服器的方法，也就是**方法導向**的服務。

而SOAP就不一樣了，SOAP是**消息導向**的服務。為什麼有了RPC卻還要開發SOAP呢? 方法導向有啥不好嗎?

也沒有不好，但是因為RPC是基於方法調用，所以難免會跟特定的程式語言耦合，也就是用戶端與伺服器端必須使用同樣的程式語言，這對開發者可以說是非常大的限制!也因為如此，基於消息的SOAP就受到許多開發者的支持，就這樣紅起來了!





