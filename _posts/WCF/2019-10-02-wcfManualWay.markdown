---
layout: post
title: 手把手教你寫WCF服務 - IIS裝載(IIS Hosting)及自裝載(Sekf-hosting) (不依賴模板)
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

所以這篇就是要來Step by Step的教學一下，如果連WCF是什麼都不知道的人記得先讀一下上面的介紹文喔。

今天會介紹兩種裝載方式，分別是*IIS裝載*及*自裝載*。

## 為什麼不依賴模板(Template)?
Visual Studio一堆模板都好好用呀!為啥不用呢? 

原因是模板通常會包入很多我們其實不需要的東西，導致整個專案過於肥大，所以我傾向不使用模板，先用最極簡的方式把服務Run起來，之後需要什麼再加就好囉!

另外還有一個很重要的原因就是，如果什麼都依賴Visual Studio強大的模板的話，其實到最後會不知道自己在幹嘛，所以初學的時候還是建議全部自己手刻一遍，等把基礎都掌握住，之後開發因為速度的因素使用模板也是OK的。

## WCF的架構
一個WCF程式由下列組態檔(Assembly)所組成，開發時也可以按照以下順序去開發:
1. *Service and Data Contract - 服務與資料契約*
2. *Services - 服務*
3. *Service Host - 服務載體*
4. *Client Proxies - 用戶端代理*
5. *Client Application - 用戶端應用程式*

接下來我們來分別看看如何實作各個組態檔吧!

### Service and Data Contract - 服務與資料契約
因為我們不使用模板的幫助，所以要自己來實作服務及資料契約。服務與資料契約呢，簡單來講就是告訴你的用戶你的服務有哪些，用的是甚麼資料型別。

首先，為了能夠使用`DataContract`、`ServiceContractAttribute`及`OperationContract`的屬性，我們必須把`System.ServiceModel`導入專案中:

導入方法:
對專案點右鍵 -> 加入參考 (Add References) -> 將`System.ServiceModel`打勾後按確定 (OK)

![](https://i.imgur.com/Xofd5hW.png)

Contracts專案組成，這邊用股票資料來當範例
* Service Contract 服務契約 : *IStockBroswer.cs* - `interface`
* Data Contract 資料契約: *StockData.cs* - `class`

![](https://i.imgur.com/KvYgety.png)

**服務契約 - *IStockBroswer.cs***
```csharp
using System.ServiceModel;

namespace WCFService
{
    [ServiceContract(Name ="StockBrowser",
                    Namespace ="http://WCFService/")]
    public interface IStockBrowser
    {
        [OperationContract]
        StockData[] GetStockByID(string stockID);

        [OperationContract]
        StockData[] GetStockByDates(string dateString);
    }
}
```

每個服務契約介面名稱上面必須標註`[ServiceContract]`的屬性，裡面的方法上面必須標註`[OperationContract]`的屬性(如果沒標這方法用戶端就無法看到)。

服務契約屬性可以帶名稱及命名空間名稱，以上面的例子來說，服務的位址就會是:
```
http://WCFService/ProductBrowser/{方法名稱}
```

**資料契約 - *StockData.cs***
```csharp
using System.Runtime.Serialization;

namespace WCFService
{
    [DataContract]
    public class StockData
    {
        [DataMember]
        public string StockID { get; private set; }

        [DataMember]
        public string StockName { get; private set; }

        [DataMember]
        public decimal OpeningPrice { get; private set; }

        [DataMember]
        public decimal ClosingPrice { get; private set; }
    }
}
```

每個資料契約的類別名稱上面必須註明`[DataContract]`，類別裡的每個屬性上必須有`[DataMember]`的屬性才可以被辨認為是資料契約的一部分。
這樣契約部分就建立完成了! 接下來來看實作服務的部分。

### Services - 服務
訂定好契約之後就要來實作服務，基本上也沒什麼特別的，就是實作介面裡的方法而已。

為了擴充性，我選擇讓服務單獨一個新的專案。

對了，為了能夠實作服務契約，記得導入`Contracts.dll`喔!

![](https://i.imgur.com/eumm2C1.png)

Service專案組成:
* Service 服務: *StockService.cs* -  `class`

![](https://i.imgur.com/JUMDW9Y.png)


```csharp
using System.Collections.Generic;
using WCFService;

namespace StockService
{
    public class StockService : IStockBrowser
    {
        public StockData[] GetStockByDates(string dateString)
        {
            List<StockData> newList = new List<StockData>();
            return newList.ToArray();
        }

        public StockData[] GetStockByID(string stockID)
        {
            List<StockData> newList = new List<StockData>();
            return newList.ToArray();
        }
    }
}
```

如上面所述服務類別只是實作服務契約裡的方法，懶得實作裡面的內容了。


### Host - 載體
下一個要實作的部分是載體，這邊我也會開一個新專案，讓每個功能都能夠獨立出來。

載體可以是*IIS應用程式*、*WAS服務*或是任何Windows的桌面應用程式(Console App、WinForm、WPF等等)。

看你想要用什麼媒介來暴露你的服務。關於載體的選擇請參照:

>[WCF (Windows Communication Foundation) 詳細介紹(一) - 什麼是WCF?(上)](https://ryanchen34057.github.io/2019/09/29/wcfIntro1/)

載體會示範兩種方式: 
1. *IIS裝載* - 走`HTTP`協定(也只能是HTTP協定)，設定檔會是`Web.config`
2. *自裝載* (Self-hosting) - 走`TCP`協定(無限制)，設定檔會是`App.config`

專案要記得把`Contracts.dll`及`Service.dll`都加入參考。

**首先是自裝載的範例**

Host專案組成:
* *Program.cs* - 視窗應用程式
* *App.config*

![](https://i.imgur.com/oZ8P1NB.png)

*Program.cs*
```csharp
public static void Main(string[] args)
{
    ServiceHost serviceHost =
       new ServiceHost(typeof(StockService));
    
    serviceHost.Open();
    Console.WriteLine(
       "Service running. Please 'Enter'
        to exit...");
    Console.ReadLine();
}
```

`App.config`
```xml
<system.serviceModel>
    <services>
        <!-- 指定服務名稱 -->
        <service name="WCFService.StockService">
            <!-- 設定端點 -->
            <endpoint
               address="net.tcp://localhost:8002/StockBrowser"
               binding="netTcpBinding"
               contract="WCFService.IStockBrowser" />
              </service>
          </services>
</system.serviceModel>
```

**再來是IIS裝載:**

IIS裝載如果不靠模板的幫助，我們要開一個空白的*ASP.net(.Net Framework)*的專案，把裡面的`Web.config`跟`.aspx`檔案清空，然後開一個空白的`.txt`檔，檔名改成`.svc`，這個svc檔是我們服務的入口。

*Host*專案組成:
* *StockService.svc* - `.svc`檔 網站入口
* *Web.config*

![](https://i.imgur.com/k6IBfEl.png)

*StockService.svc*
```xml
<%@
   ServiceHost
   Service=
   "WCFService.StockService"
%>
```

*Web.config*
```xml
<system.serviceModel>
    <services>
        <service name="WCFService.StockService">
            <!-- 注意 IIS裝載的address是空字串 !-->
            <!-- 會被IIS的虛擬目錄所取代 -->
            <endpoint
                address=""
                binding ="wsHttpBinding"
                contract ="WCFService.IStockBrowser" />
              </service>
          </services>
</system.serviceModel>
```

到這邊我們服務端的實作就完成了! 接下來是用戶端實作的部分。

### Client Proxies - 用戶端代理
代理(Proxies)是用戶端用來連接服務端及使用服務端方法的一個媒介，實現代理的方式有很多種，例如:
* *加入服務參考 (Add Service Reference)*
* *用SvcUtil*
* *自己實作Proxies類別*

因為我們這次不要依賴任何Visual Studio給我們自動產生的模板，所以我選擇自己實作Proxies的類別，之後有時間再來寫前兩種方式!

而Proxies類別也有很多種實作的方式，這次來分享實作`ClientBase`的做法以及`ChannelFactory`的作法!

這邊也要加入之前專案的`.dll`檔!

*Proxies*專案組成:
* *StockClient.cs* - `class`

![](https://i.imgur.com/TVgvzvh.png)

*StockClient.cs* (實作`ClientBase`的作法)
```csharp
using System;
using System.ServiceModel;
namespace WCFService
{
    public class StockClient: 
            ClientBase<IStockBrowser>, IStockBrowser
    {

        public StockData[] GetStockByID(string stockID)
        {
            return Channel.GetStockByID(stockID);
        }
        public StockData[] GetStockByDate(string dateString)
        {
            return Channel.GetStockByDate(dateString);  
        }
    }
}
```

*StockClient.cs* (實作`ChannelFactory`的作法)
```csharp
using System;
using System.ServiceModel;
namespace WCFService
{
    public class StockClient : IStockBrowser
    {
        public StockClient()
        {
            IStockBrowser stockBrowserChannel =
              new ChannelFactory<IStockBrowser>().
                CreateChannel();
        }

        IStockBrowser stockBrowserChannel = null;
       
        public StockData[] GetStockByID(string stockID)
        {
            return stockBrowserChannel.GetStockByID(stockID);
        }
        public StockData[] GetStockByDate(string dateString)
        {
            return stockBrowserChannel.GetStockByDate(dateString);  
        }
    }
}
```

看到這邊應該可以發現`ChannelFactory`優於`ClientBase`的地方。

就是如果我的`Proxies`的類別要實作複數個`Contracts`，因為C#的類別只能繼承一個母類別，所以我們的`ClientBase`能實作的契約最大上限就是1，所以這情況如果要多實作一個契約就只能新建一個類別。

而`ChannelFactory`的作法則可以在同一個類別裡實作無限多個契約，假設我今天除了`IStockBrowser`以外又多了一個`IStockUpdater`的契約，我們只要改成:
```csharp
using System;
using System.ServiceModel;
namespace WCFService
{
    public class StockClient : IStockBrowser, IStockUpdater
    {
        public StockClient()
        {
            IStockBrowser stockBrowserChannel =
              new ChannelFactory<IStockBrowser>().
                CreateChannel();
            IStockUpdater stockUpdater = 
              new ChannelFactory<IStockUpdater>().
                CreateChannel();
        }

        IStockBrowser stockBrowserChannel = null;
        IStockUpdater stockUpdater = null;
                   // - 以下省略 --
    }
}
```

### Client Application - 用戶端應用程式
用戶端應用程式的實作方式就自行發揮了，要修改的只有專案的`App.config`。

```xml
<system.serviceModel>
    <client>
        <!-- 理論上一個契約不能有兩種Binding，這邊只是示範，實際上會紅字 -->
        <endpoint 
            address="net.tcp://localhost:8002/
                    StockService"
            binding ="netTcpBinding"
            contract ="WCFService.IStockBrowser" />
        <endpoint 
            address="http://localhost:8002/
                    StockService.svc"
            binding ="wsHttpBinding"
            contract ="WCFService.IStockBrowser" />
          </client>
</system.serviceModel>
```

都設定完成之後，代理的使用方法:
```csharp
StockClient proxy = new StockClient();
StockData stock = proxy.GetStockByID("2330");
StockData stock = proxy.GetStockByDate("20190827");
```