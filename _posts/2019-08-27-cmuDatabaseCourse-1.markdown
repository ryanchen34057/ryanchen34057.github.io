---
layout:     post
title: 1. 關聯式資料庫介紹"    
categories: [Open Course, Database, CMU]
description: some word here
keywords: 資料庫, Database, Open Course, CMU, SQL
---

## 1. 關聯式資料庫介紹
一個有系統性，且互相關聯資料的聚集，模擬一些現實世界裡的狀況。 資料庫是所有軟體最重要的基礎。

假設我們要建構一個數位音頻的資料庫來記錄歌手及唱片的資料。

我們會需要:
```
1. 關於歌手的資訊
2. 該歌手推出了那些專輯
```
**作法1:**

>把資料庫以CSV的格式存起來，再寫程式去管理。

→ 每個實體(entity)用一個文件儲存

→ 我們的程式在每次讀寫/更新都要重新parse文件 - 頻繁I/O = 效能極差!

範例如下:
```
//Artist(name, year, country)

"Wu Tang Clan", 1992, "USA"
"Notorious BIG", 1992, "USA"
"Ice Cube", 1989, "USA"
```
Python Code:
```python
for line in file:
    record = parse(line)
    if "Ice Cube" == record[0]:
        print(int(record[1]))
```
**這樣的作法會有什麼問題呢? 可以從以下三個面向去探討**

**1. 資料完整性 (Data Integrity)**

我們要如何確保每個專輯的歌手欄都是一樣的? (如果歌手改名了，那專輯的歌手欄每個都會被更新嗎?)
如果有人把專輯的推出年分改成奇怪的字串要怎麼辦?
如果有些專輯是有複數個歌手，那CSV要怎麼存?

**2. 實作 (Implementation)**

你要如何找到某個特定的資料?
如果我們現在要撰寫一個新的軟體，要如何才可以使用同個資料庫? (其中一個軟體更動了資料庫裡的資料，那另一個也要確保會跟著改動)
如果有兩個執行緒嘗試讀取同個檔案?

**3. 資料安全性 (Durability)**

如果程式在更新資料時，主機掛掉了?
如果我們想要複製資料庫到其他主機上?


**所以... 我們需要更好的作法!!!**

總而言之，我們不只需要一個地方可以儲存資料，而且這個地方必須可以**確保每個列的格式一致**、**不同實體之間可以有關聯性(歌手 -> 唱片)**、**可以輕鬆地做到創建實體、插入、更新、刪除資料這些動作**，並且在**我們主機掛掉時可以確保傳輸中的資料不會遺失!!!**

我們需要的是.... **資料管理系統!!!**

### 資料庫管理系統 (Database Management System)
DBMS是一款軟體，讓應用程式可以在資料庫裡儲存及分析資訊。
一般的DBMS都可以做到資料庫的
1. 定義(Definition)
2. 創建(Creation)
3. 請求(Query)
4. 更新(Update)
5. 管理(Administration)

## 2. 資料模型(Data Models)
**資料模型(Data Model):** 為資訊系統提供資料的定義和格式。也就是說你的系統是如何儲存資料、如何存取資料的，然後格式是什麼，把規格訂出來比較好辦事。
資料庫的資料模型又分為以下幾種:
* **關聯式 Relational - SQL - 本文介紹的模型**
* 鍵/值 Key/ Value - NoSQL
* 圖 Graph - NoSQL
* 文本 Document - NoSQL
* 列家族 Column-family - NoSQL
* 陣列/矩陣 Array/Matrix - Machine Learning
* Hierarchical - 沒人在用了QQ
* Network - 沒人在用了QQ

接下來介紹一下關聯式模型: 
### 關聯式模型 (Relational Model)
每個關聯式資料庫裡的**實體(Entity)**稱為**表(Table)**，表以**列(Column)**及**行(Row)**組成。列及行又可以稱為**屬性(Attribute)**和**值組(Tuple)**。

假如以之前的唱片-歌手的資料庫例子來說:

值組包含有關特定歌手的所有資訊：名稱、出道年分、國籍等等。
在關聯型資料庫當中一個表就是一個**關聯(Relation)**，一個關聯式資料庫可以包含多個表。

關聯式資料庫重要用語:

* 架構(Structure)
資料內容及關連的定義。

* 完整性(Integrity)
確保資料庫的資料滿足一些約束。

* 操作(Manipulation)
如何存取及更動資料。

## 3. 關係鍵(Keys)
關聯式資料庫主要是藉由參照(Reference)的方式來達到關聯。

要讓兩個表有關聯，首先其中一個表必須要設定**主鍵(Primary Key)**，這個主鍵就是別的表可
以拿來參照的東西! 而要參照別的表的資料，就必須設定**外來鍵(Foreign Key)**。

以下為詳細的介紹:

### 主鍵(Primary Key)
 資料庫表中對儲存資料物件予以唯一和完整標識的資料列或屬性的組合。
 
 一個資料表只能有一個主鍵，且主鍵的取值不能為空值（Null）。

在DBMS裡主鍵的自動產生方式:
```
SEQUENCE /*SQL:2003*/
AUTO_INCREMENT /*MySQL*/
```

### 外來鍵(Foreign Key)
 在關聯式資料庫中，每個資料表都是由關聯來連繫彼此的關係，父資料表（Parent Entity）的主鍵（primary key）會放在另一個資料表，當做屬性以建立彼此的關聯，而這個屬性就是外來鍵。

![](https://i.imgur.com/UPgTViR.png)

### 複合主鍵 Composite Key
當單個key無法獨特地定義row，必須要用到複數個key來定義。
如下圖: 單獨靠branch_id或是supplier_name都無法獨特定義，唯有兩個綁在一起時才可。