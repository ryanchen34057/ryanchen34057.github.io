---
layout:     post
title:      "4. 資料庫儲存 I"
subtitle:   "CMU Database System Note"
date:       2019-08-28
author:     "Ryan"
header-style: text
catalog:    true
tags:
    - Open Course
    - Database
    - CMU
---
[參考來源](https://www.youtube.com/watch?v=uuX4PQXBeos&list=PLSE8ODhjZXja3hgmuwhf89qboV1kOxMx7&index=3)

# 1. 硬碟導向結構(Disk-oriented Architecture)
DBMS資料庫主要的儲存位置會是在非揮發性記憶體裡(Non-volatile memory),
也就是硬碟。
而DBMS的組件則會幫助管理資料在硬碟(非揮發性)及快取和記憶體(揮發性)之間的傳輸

## 儲存階層
![](https://i.imgur.com/j9VKk6E.png)

## 存取時間
![](https://i.imgur.com/8SuMYxW.png)


## 系統設計目標
我們的DBMS系統設計目標就是讓DBMS**可以管理超出本身記憶體容量範圍的資料庫』**。
因為將讀資料及寫資料(I/O)至硬碟非常耗效能，所以必須小心管理才不會發生嚴重的延遲及效能耗損。

資料的存取又分為兩種:
* 循序存取
* 隨機存取

以下為兩種存取方式的比較:
### 循序存取 vs 隨機存取 (SEQUENTIAL vs RANDOM ACCESS)
因為在硬碟上做隨機存取的動作比循序存取還要慢得多，所以傳統的DBMS在設計上是將循序存取最大化
* 設計演算法減少對隨機分頁的寫入，盡量讓資料以連續性的區塊儲存(contiguous block)
* 範圍(extent)即管理空間中的基本單位。 一個範圍是 8 個連續實體分頁，也就是 64 KB。

# 2. 檔案儲存(File Storage)
DBMS是將資料庫以一個或多個檔案的形式儲存於硬碟中，並且作業系統完全無法辨識這些檔案!
每個DBMS裡會有一個**儲存管理員(Storage Manager)**，負責將資料庫裡的檔案整理成一頁一頁的**分頁(Pages)**。

## 儲存管理員(Storage Manager)
負責管理資料庫的檔案
儲存管理員將檔案整理成一堆分頁(page)

另外還會負責:
* 追蹤針對分頁讀/寫的資料
* 追蹤可用的空間

## 資料庫分頁(Database Pages)
* 分頁為一個固定大小的資料的區塊
* 包含了Tuple, meta-data, indexes, log records....
* 大部分的系統不會將不同類型的分頁混再一起(存index的分頁跟存tuple的不會混在一起)
*  一些系統讓分頁可以獨立運作(self-contained)，意思是讓每個分頁都存有如何讀取跟解讀該分頁資料的資訊，讓檔案更安全(只有Oracle有)
* 每個分頁有一個獨特的ID(unique identifier)
* DBMS使用一個間接媒介(indirection layer)來對照分頁id到實體位置

### 分頁儲存架構(Page Storage Architecture)
不同的DBMS對於管理硬碟裡的檔案分頁會有不同的方式

大致上可分為以下三種:
* Heap File Organization - 無排序, 速度較快
* Sequential / Sorted File Organization - 有序, 速度較慢
* Hashing File Organization

因為第一種最常用，這邊先介紹 Heap File Organization這種儲存架構。

# 3. 資料庫堆積(Database Heap)
檔案堆積(Heap File)指沒有順序的一堆分頁，其中的tuple以隨機的順序儲存

檔案管理員所需要的API有:
* Get / Delete Page
* 必須支援可以瀏覽所有分頁的功能<br>

需要meta-data去記錄存在那些分頁還有剩餘的空間

### 兩種用來定義heap file的方式
* Linked List
* Page Directory

### Linked List (BAD)
在每個檔案的開頭存一個header page，裡面存兩個指標(pointer)
→  空的分頁列表的頭(HEAD of free page list)
→  資料分頁列表的頭(HEAD of data page list)
每個分頁會自己記錄剩餘的空間(free slots)

![](https://i.imgur.com/bU0iQ5C.png)

### Page Directory (BETTER)
DBMS另外管理一個特別的分頁用來記錄資料分頁在資料庫檔案裡的位置
Directory也會記錄所有分頁裡有的剩餘的空間(free slots)

DBMS必須確保Directory分頁與其他分頁的資訊是一致的

![](https://i.imgur.com/xL2ZPoS.png)

### 分頁頭(Page Header)
每個分頁裡都含有一個頭(Header)用來存放此分頁的內容資訊
* 分頁大小(Page Size)
* 校驗碼(CheckSum)
* DBMS version
* 交易透明性(Transaction Visibility)
* 壓縮資訊(Compression Information)

某些系統讓分頁是可以獨立運作的(ex. Oracle)

![](https://i.imgur.com/aVJKJnt.png)


# 4. 分頁佈局(Page Layout)
分頁佈局可以分為兩種方式:
* 值組導向(Tuple-oriented)
* 紀錄導向(Log-oriented)
  
## 值組導向(Tuple-oriented)
要如何存tuple在分頁裡?
### BAD IDEA:

在每個分頁裡記錄Tuple的數量，然後將新的Tuple插入最後面
→ 但如果刪除中間的Tuple?
→ 如果Tuple裡的Attribute有變數?

![](https://i.imgur.com/yg0Owgp.jpg)

### GOOD IDEA: Slotted Pages
最常見的佈局方式，用一個Slot Array對應每個Tuple的位置的offset

Header紀錄: Slot的數量、最後使用的slot位置 - 開始位置

![](https://i.imgur.com/7PUggsq.png)


## 紀錄導向(Log-structured)
DBMS只存入Log Record而不是Tuple

系統存入資料庫如何被更動的紀錄(Log record)到檔案裡
* INSERT儲存整個Tuple
* DELETE紀錄被刪除的Tuple
* UPDATE含有被更動過的Attribute的變化值(delta)

![](https://i.imgur.com/WUuMFZf.png)

要讀取一個紀錄，DBMS從後面開始瀏覽log，然後重新建構(recreate)Tuple來找尋目標

![](https://i.imgur.com/NW6tgnp.png)

創建Indexes會提升效能因為可以直接跳到該位置

![](https://i.imgur.com/p1xdGsV.png)

### Log-structure Compaction
Compaction利用刪除不必要的紀錄來讓巨大的log檔案變小
* Level Compaction
  
![](https://i.imgur.com/6xMnI1x.png)

* Universal Compaction
  
![](https://i.imgur.com/3AuiuQ8.png)

# 5. 值組佈局(Tuple Layout)
Tuple只是一串Bytes，DBMS要負責將它編譯成屬性的型別以及值

## 值組頭(Tuple Header)
每個Tuple都會有一個Header，含有其meta-data的:
* 可見度資訊(Visibility info)
* NULL值的BitMap
  
※不需要存Schema的meta-data

![](https://i.imgur.com/auGqOXR.png)

## 反正規化 Denormalized Tuple Data
簡單來講就是將有相關聯的Tuple盡量結合在同一個分頁裡

→ 對一般的workload來說，有機會減少I/O的次數

→ 但會讓UPDATE更耗效能

※要注意:
* 若要以查詢(select)效能為考量，必須進行適當的反正規化
* 在進行資料表的反正規化之前，必須徹底了解系統中有哪些功能很頻繁地修改(update)資料表，如果系統功能很頻繁地update資料表，反正規化資料表後會影響update的效能，因為欲update的資料可能是很多筆，例如，將客戶資料表裡國籍編號為TPE的客戶update成國籍編號為TAIWAN，國籍編號為TPE的客戶可能數萬筆以上，實是造成效能上的影響。 

## Record ID
DBMS需要一種方式來記錄Tuple

每個Tuple會被分配一個獨特的Record ID

→ 最常見: page_id + offset / slot

→ 也可以包含文件的位置資訊

# 6. MySQL資料庫三大正規化及反正規化
## 第一正規化 (1NF)
資料表中的每一列(欄位)，必須是不可拆分的最小單元，也就是確保每一列的原子性(atomicity)。例如如果有個屬性是地址+電話，那就違反了第一正規化的規則。

## 第二正規化 (2NF)
滿足1NF後要求表中的所有列，都必需依賴於主鍵，而不能有任何一列與主鍵沒有關係（一個表只描述一件事情）。

例如：
>訂單表只能描述訂單相關的資訊，所以所有的欄位都必須與訂單ID相關。
產品表只能描述產品相關的資訊，所以所有的欄位都必須與產品ID相關。
因此在同一張表中不能同時出現訂單資訊與產品資訊。

## 第三正規化 (3NF)
滿足2NF後，要求表中的每一列都要與主鍵直接相關，而不是間接相關（表中的每一列只能依賴於主鍵）

例如：

>一個使用者可以對應多個角色，一個角色也可以對應多個使用者。則可以按如下方式建立資料表關係，使其滿足第三正規化。

### user
```
+-----+----------+----------+
| uid | username | password |
+-----+----------+----------+
|   1 | RYAN     | 123      |
|   2 | ALISA    | 246      |
|   3 | BEN      | 789      |
+-----+----------+----------+
```
### role
```
+---------+--------+
| role_id | name   |
+---------+--------+
|     100 | Knight |
|     200 | Witch  |
+---------+--------+
```
### user_role
```
// 使用者可以有多個角色，角色也可以有多個使用者
+----+------+---------+
| id | uid  | role_id |
+----+------+---------+
|  1 |    1 |     200 |
|  2 |    1 |     100 |
|  3 |    2 |     100 |
|  4 |    3 |     200 |
+----+------+---------+
```

## 反正規化(Denormalize)
反正規化化指的是通過增加冗餘或重複的資料來提高資料庫的讀效能。
```
+----+------+---------+-----------+
| id | uid  | role_id | role_name |
+----+------+---------+-----------+
|  1 |    1 |     200 | Witch     |
|  2 |    1 |     100 | Knight    |
|  3 |    2 |     100 | Knight    |
|  4 |    3 |     200 | Witch     |
+----+------+---------+-----------+
```
例如：在上例中的user_role使用者-角色中間表增加欄位role_name。
反正規化化可以減少關聯查詢時，join表的次數。