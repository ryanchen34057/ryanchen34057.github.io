---
layout: post
title: 手把手教你用WCF部屬網路服務 (不依賴模板)
categories: [WCF]
description: 
keywords: WCF, Windows, Windows Communication Foundation, WCF是什麼
---

## 前言
前幾天寫了[Programming WCF Services](http://shop.oreilly.com/product/0636920032373.do)的筆記，大概把WCF的介紹拆成三篇文:

[WCF (Windows Communication Foundation) 詳細介紹(一) - 什麼是WCF?(上)](https://ryanchen34057.github.io/2019/09/29/wcfIntro1/)

[WCF (Windows Communication Foundation) 詳細介紹(二) - 什麼是WCF?(中)](https://ryanchen34057.github.io/2019/09/29/wcfIntro2/)

[WCF (Windows Communication Foundation) 詳細介紹(三) - 什麼是WCF?(下)](https://ryanchen34057.github.io/2019/09/29/wcfIntro3/)

寫完之後自己看了一下發現，第一次看的人可能全部看完還是不知道WCF怎麼部屬服務阿XD

所以這篇就是要來Step by Step的教學一下，如果連WCF是什麼都不知道的人記得先讀一下上面的介紹文喔!

## 為什麼不依賴模板(Template)?
Visual Studio一堆模板都好好用呀!為啥不用呢? 原因是模板通常會包入很多我們其實不需要的東西，導致整個專案過於肥大，所以我傾向不使用模板，先用最極簡的方式把服務Run起來，之後需要什麼再加就好囉!

另外還有一個很重要的原因就是，如果什麼都依賴Visual Studio強大的模板的話，其實到最後會不知道自己在幹嘛，所以初學的時候還是建議全部自己手刻一遍，等把基礎都掌握住，之後開發因為速度的因素使用模板也是OK的。

## WCF的架構
一個WCF程式由下列元件(Assembly)所組成，開發時也可以按照以下順序去開發:
1. *Service and Data Contract - 服務與資料契約*
2. *Services - 服務*
3. *Service Host - 服務載體*
4. *Client Proxies - 用戶端代理*
5. *Client Application - 用戶端應用程式*

接下來我們來分別看看每個元件的實作吧!

### Service and Data Contract - 服務與資料契約
因為我們不使用模板的幫助，所以要自己來實作服務及資料契約。

首先，為了能夠使用`DataContract`、`ServiceContractAttribute`及`OperationContract`的屬性，我們必須把`System.ServiceModel`導入專案中:

對專案點右鍵 -> 加入參考 (Add References) -> 將`System.ServiceModel`打勾後按確定 (OK)

![](https://i.imgur.com/Xofd5hW.png)

Contracts專案組成，這邊用股票資料來當範例
* Service Contract 服務契約 : *IStockBroswer.cs* - `interface`
* Data Contract 資料契約: *StockData.cs* - `class`

![](https://i.imgur.com/AZr215X.png)

*IStockBroswer.cs*
```csharp
using System.ServiceModel;

namespace WCFService
{
    [ServiceContract(Name = "ProductBrowser",
                     Namespace = "http://WCFService")]
    public interface IProductBrowser
    {
        [OperationContract]
        StockData GetStockByID(string stockID);

        [OperationContract]
        StockData[] GetStockByDate(string dateString);
    }
}
```

每個服務契約介面名稱上面必須標註`ServiceContract`的屬性，裡面的方法上面必須標註`OperationContract`的屬性(如果沒標這方法用戶端就無法看到)。

*StockData.cs*
```csharp
using System.Runtime.Serialization;

namespace WCFService
{
    [DataContract]
    public class StockData
    {
        private string _stockID;
        private string _stockName;

        [DataMember]
        public string StockID
        {
            get { return _stockID; }
            set { _stockID = value; }
        }

        [DataMember]
        public string StockName
        {
            get { return _stockName; }
            set { _stockName = value; }
        }
    }
}

```