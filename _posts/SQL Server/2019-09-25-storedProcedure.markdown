---
layout: post
title: SQL Server 預存程序(Stored Procedure)用法實例
categories: [SQL Server]
description: Stored Procedures
keywords: SQL, SQL Server, 預存程序, Stored Procedures
---

## 什麼是預存程序(Stored Procedure)?
預存程序是SQL Server的一個功能，能夠先存下一段SQL的指令以便之後使用。有點類似於把一個函數存在一個變數裡，之後呼叫這個變數的時候就可以跑裡面的程式碼。

## 用法
1. 打開SQL Server Management Tool之後展開資料庫(範例是用AdventureWorks的Sample)，在可程式性(Programmibility)底下的預存程序按右鍵，點選預存程序之後應該會打開一個頁面，裡面有一些預存程序該如何撰寫的範例。

![](https://i.imgur.com/TKyTEMG.png)

2. 我們這邊從頭開始示範，所以可以先把內容刪掉。輸入以下指令
```sql
/*創建預存程序語法: CREATE PROCEDURE {程序名稱}*/
CREATE PROCEDURE dbo.spGetAllPeople /*注意: 名稱不能是sp_開頭!("sp_"是預留給系統的))*/
AS
BEGIN
    /*從這邊開始輸入要預存的SQL指令*/
    SELECT * FROM Person.Person; /*Person.Person -> 資料表名稱*/
END 
```
![](https://i.imgur.com/ou9xPnS.png)

按執行之後就完成了最基本的預存程序!

3. 如果要在SQL Management Tool裡調用預存程序，要輸入以下指令:
```sql
EXEC dbo.spGetAllPeople
```
※注意: 如果一開始會出現紅字，代表暫存裡尚未更新，
這時候只要`ctrl + shift + R`就可以刷新頁面了!

4. 之後如果要修改，要用`ALTER`而不是`CREATE`，範例如下:
```sql
ALTER PROCEDURE dbo.spGetAllPeople /*注意: 名稱不能是sp_開頭!("sp_"是預留給系統的))*/
AS
BEGIN
    /*這邊要改成不要讓程序每次結束都回傳影響了多少row的訊息*/
    SET NOCOUNT ON;

    /*從這邊開始輸入要預存的SQL指令*/
    SELECT * FROM Person.Person; /*Person.Person -> 資料表名稱*/
END 
```

5. 如果要傳入變數的話要怎麼做呢?
```sql
CREATE PROCEDURE dbo.spPeople_GetLastName
    @LastName nvarchar(50) /*屬性的型別要記得傳入*/
AS
BEGIN
    SELECT FirstName, LastName FROM Person.Person
    WHERE LastName = @LastName;
END
```

執行

![](https://i.imgur.com/luewTmK.png)

結果

![](https://i.imgur.com/hs9WLZ0.png)

或是也可以像這樣指定兩個變數:
```sql
CREATE PROCEDURE dbo.spPeople_GetLastName
    @LastName nvarchar(50) 
    @FirstName nvarchar(50)
AS
BEGIN
    SELECT FirstName, LastName FROM Person.Person
    WHERE LastName = @LastName AND FirstName = @FirstName;
END
```

## 預存程序的威力
假設我們的使用者端可以存取到我們的資料庫，但是我們又不想暴露太多資料庫的資訊或是想要降低被竊取資訊的可能性，這時候我們可以再創建一個使用者的角色(Role)，而這個角色就只能做特定的預存程序。

實際做法如下:

1. 先創建一個新的角色， 打開一個新的查詢，輸入:
```sql
/*新增一個叫做dbStoredProcedureOnlyAccess的角色*/
CREATE ROLE dbStoredProcedureOnlyAccess
```

執行並刷新頁面後在安全性 -> 角色 -> 資料庫角色的底下就可以看到我們剛剛新增的角色。

![](https://i.imgur.com/uBmD14D.png)

2. 為了要讓新角色可以使用預存程序，我們要給予他使用`execute`的權限
```sql
GRANT EXECUTE TO dbStoredProcedureOnlyAccess
```
執行後該角色的使用者就可以開始使用預存程序了!

3. 接下來創建一個新的登入帳號，讓該帳號的權限僅限於`dbStoredProcedureOnlyAccess`的權限

從伺服器下面的安全性 -> 登入按右鍵新增登入到達以下畫面

![](https://i.imgur.com/aAKodY6.png)

4. 輸入登入名稱，選擇SQL Server驗證，輸入密碼。(這邊可以不勾選強制執行密碼原則，這樣密碼的設定就可以不受限制)

![](https://i.imgur.com/VD2qCcE.png)


5. 伺服器角色維持預設的public，使用者對應這邊選擇資料庫之後，在下面的角色成員資格對象只勾選`public`及`dbStoredProcedureOnlyAccess`

![](https://i.imgur.com/Bl65GZy.png)


這樣我們就創建好一個帳號只有使用預存程序的權限了!

>參考: https://www.youtube.com/watch?v=Sggdhot-MoM&list=PLLWMQd6PeGY0qGAge91kAaAo-YlFD_0dP&index=7