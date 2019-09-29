---
layout: post
title: 淺談Web Service(一) - SOA(Service-Oriented Architecture)
categories: [Web Service]
description: SOA(Service-Oriented Service)
keywords: Web Service, SOA, Rest
---

## 前言
在以前還沒有網路的時代，所有的工作都在一台機器底下完成，工程師自然不需要擔心自己所寫的程式要如何跟別台機器溝通。

但隨著網路的出現，系統之間開始需要通信，但是不同機器的架構不同，語言也不同，那要怎麼溝通呢?

**這時候SOA(Service-Oriented Architecture)就出現了!**


## SOA介紹
其實SOA就是一種概念，工程師把軟件細分成一個個組件，每個組件包裝成服務，服務可在不同系統之間調用。所以SOA就是一種以服務為導向的架構! (Service-Oriented Architecture)

SOA的架構厲害之處就是可以讓不通用的系統或是應用來通過某些協議來達到相互溝通，因此SOA獨立於任何的技術或是產品，SOA的服務提供簡單操作的接口，讓開發者能夠輕鬆地使用服務，而不需要了解該服務複雜的底層運作。

SOA可以比喻為插座，我把電力的供給做成一個插座(服務接口)，如果你要用電就用插頭(規範好的協議))向我連接，身為使用者的你不需要知道插頭裡面的電線是如何運作的，透過接口就可以輕易地取得電力。

而關於SOA的服務，常見的設計原則有:
1. 必須是無狀態(Stateless)，也就是避免請求者必須依賴提供者狀態這種情況。
> 延伸閱讀: [Stateful vs Stateless 兩者差異比較](https://ryanchen34057.github.io/2019/09/28/statefulAndStateless/)

2. 一個服務一個功能
3. 接口有明確的定義，例如`XML`或是現在比較主流的`JSON`格式

因為SOA架構的實現概念不依賴於單一技術，所以可以被各種不同技術實現，而**Web Service**就是其中一種實現方式。

## Web Service
**Web Service**就是定義服務端如何向客戶端提供服務的方法。

常見的方法有下列幾種:
1. **RPC (Remote Procedure Call)** - 稱為遠程方法調用協議，為方法導向
2. **SOAP (Simple Object Access Protocol)** - 稱為簡單物件存取協議，為訊息導向
3. **REST (Representational State Transfer)** - 稱為表象化狀態轉變，為資源導向

關於每種Web Service的介紹請期待下集XD