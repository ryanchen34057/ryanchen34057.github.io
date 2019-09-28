---
layout: post
title: 淺談Web Service(二) - RPC 遠程方法調用
categories: [Web Service]
description: RPC, SOAP, REST
keywords: Web Service, SOA, RPC, SOAP, REST
---

上集: [淺談Web Service(一) - SOA(Service-Oriented Architecture)](https://ryanchen34057.github.io/2019/09/28/webServiceIntro/)

這篇來介紹Web Service常見的作法有哪些吧!

1. **RPC**: 即為遠程方法調用(Remote Procedure Call)，概念為**用戶端去調用服務端的方法(服務)**。這裡的用戶端通常是會向遠端系通呼叫的應用程式，透過向裝置RPC協定的伺服器發出HTTP請求。

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

         // 向伺服器發送請求，方法sum(17, 13)在伺服器端被呼叫，回傳的一定會是Object的物件
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

### XML-RPC 伺服器端
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

伺服器端接收到用戶端的方法調用請求後，在計算出結果後會回傳以下的XML:
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

以上即為一個簡單的XML-RPC實作範例，XML-JSON的實作方式其實也差不多，只是傳送格式不同。
有興趣的可以自己試試看!

> 參考來源:
> 
> https://zhuanlan.zhihu.com/p/28858632
> 
> https://www.tutorialspoint.com/xml-rpc/xml_rpc_examples.htm