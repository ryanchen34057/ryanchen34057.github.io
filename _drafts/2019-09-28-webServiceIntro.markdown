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
