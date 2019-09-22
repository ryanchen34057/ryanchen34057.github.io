---
layout:     post
title: 7. 雜湊表 Hash Table
categories: [Database]
description: some word here
keywords: 資料庫, Database, Open Course, CMU, SQL
---

[參考來源](https://www.youtube.com/watch?v=ByG1IMM6Ua8&list=PLSE8ODhjZXja3hgmuwhf89qboV1kOxMx7&index=6)

# 1. 設計選擇 Design Decisions
## 資料管理 Data Organization
→ 我們要如何設計記憶體/分頁裡的資料結構，以及要存放什麼資訊才能讓資料存取更有效率

## 並發性 Concurrency
→ 如何設計才能夠讓資料庫在多個執行緒同時間存取同一個資料的狀況下不會出問題

# 2. 雜湊表(Hash Table)
### 設計選擇#1: Hash Function
→ 如何將巨大的鍵空間對應到較小的區域

→ 速度快 vs. 碰撞率的抉擇
### 設計選擇2: Hashing Scheme
→ 如何處理鍵碰撞

→ 分配更大的hash table vs. 額外的鍵插入/搜尋機制的抉擇

## Hash Functions
建議不要使用加密類的Hash Function，因為這是在我們內部系統裡使用，所以不用在意安全性。
我們需要的是一個**速度快**且**碰撞率低**的Function。

* MurmurHash (2008)
  
→ 一種快且用途廣泛的哈希函數
* Google CityHash (2011)
  
→ 從MurmurHash2衍生而來

→ 鍵的長度很短(<64 bytes)，但是速度卻很快
* Google FarmHash (2014)
  
→ 新版本的CityHash，碰撞率更低
* CLHash (2016)
  
→ 基於carry-less multiplication的快速哈希函數

## Static Hashing Schems
當發生碰撞時，須採取的措施
### 作法#1: Linear Probe Hashing
一發生碰撞就繼續往下找，直到找到空的slot為止。

→ 如想知道某資料是否存在，只要hash要某個位置的index，然後再往下找

→ 必須將key存在index裡，這樣才能知道什麼時候可以停止掃描

→ 插入及刪除都是一般的查找而已
### 處理Non-unique keys的問題
選擇#1: Seperate LinkedList
→ 將每個key的值存放在不同的記憶空間中
![](https://i.imgur.com/mM8KxvT.jpg)
 
選擇#2: Redundant Keys(較常用)
→ 把重複的key一起存在hash table裡
![](https://i.imgur.com/TxUztja.jpg)


#### 優點
速度快
#### 缺點
刪除需要做一次rehashing，所以效能會比較差

### 作法#2: Robin Hood Hashing
一種Linear Probe Hashing的變化版，將"rich" key的空間搶來，然後給"poor" key
1. 插入A
   
![](https://i.imgur.com/U1xuuo5.jpg)

2. 插入B
   
![](https://i.imgur.com/k1XVRBK.jpg)

3. 插入C，但是跟A碰撞
   
![](https://i.imgur.com/CVo5GpZ.jpg)

4. 插入D，但是與C碰撞
   
![](https://i.imgur.com/4cmYblJ.jpg)

5. 插入E，但是與A碰撞，往下找之後又與C碰撞，到了D時，因為E的步數比D多，所以E把D的位置給搶走
   
![](https://i.imgur.com/ZZThb9i.jpg)

### 作法#3: Cuckoo Hashing
用複數個hash table及不同的hash function

→ 插入時，確認所有table後選擇有剩餘空間的

→ 如果沒有table有剩餘空間時，從其中一個table騰出空間，然後重新hash一次找到新的位置

查找及刪除都是O(1)因為每個hash table只有一個位置會被確認到。
1. 首先插入A，然後分別用兩種Hash Function，因為兩個table都是空的，所以插入隨便一個
   
![](https://i.imgur.com/EqKKmgc.jpg)

2. 再來插入B，hash1只到A的位置，所以就轉存到table2的位置
   
![](https://i.imgur.com/wzN4yWR.jpg)

3. 插入C，hash1到A的位置，hash2到B的位置
   
![](https://i.imgur.com/iunlN6P.jpg)

4. 隨機的選了B，將B的空間奪走 
   
![](https://i.imgur.com/vJysFdr.jpg)

5. 再用hash1一次B，到了跟A一樣的地方，所以把A的空間奪走
   
![](https://i.imgur.com/nu1EhoQ.jpg)

6. 再用hash2一次A，插入table2
   
![](https://i.imgur.com/J9FnzzL.jpg)

#### ※注意
需確認沒有卡在無限迴圈裡

如果發現卡在無限迴圈，就會需要重建一個hash table及hash function。

→ 如果有兩個hash function，在table超過50%滿時重建的機會比較大

→ 如果有三個hash function，在table超過90%滿時重建的機會比較大


**這些hash table都需要事先知道你要存放多少element**

→ 不然在需要變大或是縮小都要重新建構整個table一次

而Dynamic Hash Table可以隨時變大或是縮小

→ Extensible Hashing

→ Linear Hashing


### Chained Hashing
讓hash table裡的每個slot都存一個linked list

→ 要查看某個element是否存在，只要跑一次hash然後掃描即可

→ 插入及刪除都是一般的查找而已

→ hash table可以無限成長
![](https://i.imgur.com/smyUxKy.jpg)

#### 缺點
因為buckets會無限擴張，所以比較難做一些交換及更動，不然就要重新建構一個hash table

### Extendible Hashing
Chained Hashing的進化版。Extendable Hashing 的用意在於，由於Collision無法避免（尤其在 Hash Value 被縮限過的情形下），於是轉而設法降低往 Bucket 搜尋時的成本。
並且在擴大的時候，變動是局部性的。
設計上在 Bucket 前加入一層 Directory ，每當有新的 Index 加入時，針對其 Hash Value 取其 Binary Value。

* Global Depth: 分辨 Index 所屬 Bucket 時需要的最大Bit數
* Local Depth: 分辨 Index 是否屬此 Bucket 時需要的Bit數
![](https://i.imgur.com/mgzwTHZ.jpg)

Find A，找到第一個Bucket
![](https://i.imgur.com/WPTQk1q.jpg)

Insert B，找到第二個Bucket
![](https://i.imgur.com/mcV6JbP.jpg)

Insert C，只到第二個Bucket，但是滿了，所以要做split
![](https://i.imgur.com/RkTQS5Y.jpg)

split之後，Global Depth變3，insert C(注意這邊只有滿的那個bucket有做變動)
![](https://i.imgur.com/4jE5d6g.jpg)
