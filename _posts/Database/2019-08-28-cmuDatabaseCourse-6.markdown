---
layout:     post
title:      "6. 緩存池 Buffer Pools"
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
DBMS是如何管理記憶體以及從硬碟裡搬移資料的?

這個問題牽扯到兩個面向:

### 空間控制 Spatial Control
* 要把分頁寫在硬碟的哪裡?
* 目標是把有相關聯的分頁盡可能地靠在一起

### 時序控制 Temporal Control
* 要何時將分頁寫入記憶體裡，以及何時將它們寫入硬碟中
* 目標是降低從硬碟讀取資料時的延遲數

![](https://i.imgur.com/p4Ef3UW.jpg)

# 1. Buffer Pool管理
## Buffer Pool介紹
被整理為一個存放固定大小分頁陣列的記憶體區塊。

陣列的每個格子稱為**Frame**

當DBMS請求一個分頁時，該分頁會被複製到其中一個Frame之中。(如下圖)

![](https://i.imgur.com/w7cwtyJ.jpg)

## Buffer Pool Meta-data
分頁的Hash table紀錄目前存放在記憶體裡的分頁資訊(Page ID...)

同時也記錄著關於分頁的其他資料:
* **Dirty Flag**(用一個標誌位(flag)來表示一組資料的狀態，如果設定了標誌位,那麼表示這組資料處於dirty狀態,這個時候需要重新計算或者同步)
* **Pin/Reference Counter**(看哪個page被鎖住)

Hash Table裡的page ID會對應到Buffer Pool裡的分頁:

![](https://i.imgur.com/r5lQNV8.jpg)

當有執行緒要存取某個分頁時，Page Table裡該分頁會被做記號(Pin)，被做記號後其他執行緒也可以讀取該分頁，但是Buffer Pool就不會把該分頁的資料移除，因為有人正在使用中。

![](https://i.imgur.com/UMCApbo.jpg)

當嘗試讀取沒有在Buffer Pool裡的分頁資料時，Page Table裡的空格會先被鎖住(latched)(因為怕被其他執行緒給改寫掉了)，然後去找到欲讀取的分頁資料後再更新Page Table裡的資料。

![](https://i.imgur.com/MwYYwfx.jpg)

這邊說明一下Lock與Latch的區別:

## Locks vs Latches
簡單來說Locks是用來保護資料庫裡的內容，例如表格、Tuple或是資料庫等實體(Entity)，而Latch則是保護DBMS裡的資料結構不受其他執行緒影響。
### Lockes
* 保護資料庫的內容不被其他交易影響
* 讓資料庫交易更有保障
* 必須要有回復機制
* 可以Lock表格、Tuple或是資料庫，也就是資料庫裡的實體

### Latches
* 保護DBMS裡的內部資料結構不被其他執行緒影響
* 讓操作更有保障
* 不需有回復機制

## Page Table vs Page Directory
### Page Directory
Page Directory在資料庫檔案裡，將pageID對應到分頁的所在位置。

→ **所有更動都必須存在硬碟裡，讓DBMS重新啟動後也能找得到**

### Page Table
Page Table在Buffer Pool裡，負責將pageID對應到Buffer Pool的Frame的位置，以存取分頁的複製檔。

→ **這是一個存在記憶體裡的資料結構，所以不需要存放在硬碟裡。**

## Multiple Buffer Pool
Buffer Pool不一定只會有一個，可以依照用途，分出很多的Buffer Pool
* 多個Buffer Pool的實體
* 每個資料庫(database)的Buffer Pool
* 每個分頁類型(per-page type)的Buffer Pool

多個Buffer Pool幫助降低latch的連接以及增進數據局部性。

## 掃描共享(Scan Sharing)
Query可以重複利用儲存的或是操作計算後的資料。

→ 與Result Caching不同

有個query開始scan table，如果已經有另一個在做同樣的事情，DBMS會將其與第二個query的cursor連接起來

→ Query不一定要完全一樣

→ 也可以分享即時的結果

只有大公司例如IBM DB2和MSSQL有完全支援這東西!
![](https://i.imgur.com/kEswIKF.jpg)

## Buffer Replacement Policies
當DBMS需要清空一個Frame給新的分頁的時候，它必須決定要從Buffer Pool清除哪個分頁。

目標:
* 正確性
* 準確性
* 速度
* Meta-data overhead

### 方法一: Least-Recently-Used
記錄一個Timestamp，這樣就能知道每個分頁最後被存取的時間。
然後當DBMS需要清分頁時，選擇時間最久的那個分頁。
→ 排序分頁來降低搜尋欲清除分頁時所需要花費的時間

### 方法二: Clock
類似LRU，但是不用每個分頁都記錄一個timestamp

→ 每個分頁都有一個reference bit

→ 如果分頁被存取了，將reference bit設為1

將利用"clock hand"，分頁以circular buffer的方式管理

→ 指到分頁時，確認分頁的bit是否已被設為1

→ 如果是的話，設為0。如果不是，將分頁清除。

![](https://i.imgur.com/tPjlCVH.jpg)

![](https://i.imgur.com/P8v8Mgh.jpg)

![](https://i.imgur.com/NS3793c.jpg)

### 這兩個方法會有的問題
#### Sequential Flooding!
→ 有一個Query實行全表掃描(Sequential Scan)
→ 這樣會讓Buffer Pool充滿了只讀取了一次卻沒再使用過的分頁

**最近讀取的分頁其實就是最沒用的分頁!!!**

#### 更好的方法一: LRU-K
把過去最後K個reference的歷史紀錄當作Timestamp，然後計算其之間的存取間隔多久。
DBMS就利用這個歷史紀錄來評估該分頁下一次會被存取的時間。

#### 更好的方法二: Localization
DBMS在每個query時就決定要把哪個分頁剔除。這樣減少每次query，buffer pool被汙染的機會。
→ 紀錄query存取過的分頁有哪些

#### 更好的方法三: Prioty Hints
讓DBMS知道每個分頁在query執行時的重要性
這樣可以提供一些資訊給Buffer Pool，讓它知道該分頁到底該不該踢掉

以上三種方法通常只有在大型商業化的DBMS才有提供，開源的MySQL跟Postgres等雖然已經很棒，但是沒辦法做到這麼細微的操作。

## Dirty Pages
dirty page是指在Buffer Pool中已被修改過的頁面，這些頁面由於尚未寫入disk file中，此時頁面在Buffer Pool跟disk file中的內容是不同的；所以將此頁面稱為dirty。

### Background Writing
DBMS可以定期掃描page table，然後將dirty page寫入硬碟中。
當dirty page被安全地寫入後，DBMS可以選擇看是要把分頁清除，還是把dirty flag拿掉。
這邊要特別注意我們不會在log record被寫入之前就把dirty pages寫入。