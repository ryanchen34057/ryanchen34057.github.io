---
layout: post
title: 淺談Web Service(二) - RPC 遠程方法調用
categories: [Web Service]
description: RPC, SOAP, REST
keywords: Web Service, SOA, RPC, SOAP, REST
---

上集: [淺談Web Service(一) - SOA(Service-Oriented Architecture)](https://ryanchen34057.github.io/2019/09/28/webServiceIntro/)

這篇來介紹Web Service中的RPC吧!

## RPC 遠程方法調用介紹
即為遠程方法調用(Remote Procedure Call)，概念為**用戶端去調用服務端的方法(服務)**。這裡的用戶端通常是會向遠端系統呼叫的應用程式，透過向裝置RPC協定的伺服器發出HTTP請求。

實現方式有兩種:

* [XML](https://en.wikipedia.org/wiki/XML)-RPC
* [JSON](https://en.wikipedia.org/wiki/JSON)-RPC

首先來看一下XML-RPC的實作範例:

### XML-RPC Client端
```java
import java.util.*;
// 包含XML-RPC Client及Server的Java Package
import org.apache.xmlrpc.*;

public class JavaClient {
   public static void main (String [] args) {
   
      try {
          // localhost - 本地端，也可以是一個IP位置
         XmlRpcClient client = new XmlRpcClient("http://localhost/RPC2"); 
         Vector params = new Vector();
         
         params.addElement(new Integer(17));
         params.addElement(new Integer(13));

         // 向伺服器發送請求，方法sum(17, 13)在服務端被呼叫，回傳的一定會是Object的物件
         // "sample"是伺服器的一個handler
         Object result = server.execute("sample.sum", params);

         // 將回傳的物件轉為Integer的類別，再將結果印出來
         int sum = ((Integer) result).intValue();
         System.out.println("The sum is: "+ sum);

      } catch (Exception exception) {
         System.err.println("JavaClient: " + exception);
      }
   }
}
```
上面的Java程式會向伺服器傳送下面的XML:
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<methodCall>
   <methodName>sample.sum</methodName>
   <params>
      <param>
         <value><int>17</int></value>
      </param> 
      <param>
         <value><int>13</int></value>
      </param>
   </params>
</methodCall>
```

### XML-RPC 服務端
```java
import org.apache.xmlrpc.*;

public class JavaServer { 
    // 被用戶端調用的方法
   public Integer sum(int x, int y){
      return new Integer(x+y);
   }

   public static void main (String [] args){
   
      try {

         System.out.println("Attempting to start XML-RPC Server...");
         
         WebServer server = new WebServer(80);
         // 加入可被用戶端訪問的handler
         server.addHandler("sample", new JavaServer());
         server.start();
         
         System.out.println("Started successfully.");
         System.out.println("Accepting requests. (Halt program to stop.)");
         
      } catch (Exception exception){
         System.err.println("JavaServer: " + exception);
      }
   }
}
```

服務端接收到用戶端的方法調用請求後，在計算出結果後會回傳以下的XML:
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<methodResponse>
   <params>
      <param>
         <value><int>30</int></value>
      </param>
   </params>
</methodResponse>
```

## RPC優點
1. 溝通方式非常簡單且容易實作，溝通協議容易統一。
許多大公司都有使用自己維護的RPC框架，例如Google的[gRPC](https://github.com/grpc/grpc)，使用方式非常簡單，只需要一個文件就可以搞定兩邊的溝通協議模式。

2. 現在很多RPC框架可以跨語言，所以可以用比較方便的語言撰寫測試程序，這樣測試起來也會比較簡單。(例如用`Python`模擬`C/C++`程式)

## RPC缺點
1. 溝通方式比較單一，比較不適合多系統間複雜的協議溝通。
因為RPC的描述方式較為單一，一應一答，所以有些狀況下是沒辦法使用RPC的，例如有個服務需要協調多個模組、或是伺服器需要主動發送通知給用戶端等等。

2. 例外處理比較困難。



延伸閱讀:
[淺談Web Service(一) - SOA(Service-Oriented Architecture)](https://ryanchen34057.github.io/2019/09/28/webServiceIntro/)



> 參考來源:
> 
> https://zhuanlan.zhihu.com/p/28858632
> 
> https://www.tutorialspoint.com/xml-rpc/xml_rpc_examples.htm