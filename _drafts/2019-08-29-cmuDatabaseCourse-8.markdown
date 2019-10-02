---
layout:     post
title: 8. 樹索引 Tree Index
categories: [Database]
description: some word here
keywords: 資料庫, Database, Open Course, CMU, SQL
---

[參考來源](https://www.youtube.com/watch?v=Z1Qrsm7EfRw&list=PLSE8ODhjZXjYutVzTeAds8xUt1rcmyT7x&index=10)
# 1. 表格索引 Table Indexes
表格索引(Table Index)有點像是把表格裡的列的一小部分複製並將其按照順序整理起來，讓存取更有效率。
而DBMS則確保表格裡的內容與index是一致的。(像是書裡的Glossary)

**選擇什麼query要用什麼index是DBMS的工作!!!**

# 2. B+TREE
看不懂就看這裡: 
[介紹1](https://www.youtube.com/watch?v=IcDMCoyPFG0)
[介紹2](https://www.kancloud.cn/kancloud/theory-of-mysql-index/41844)

B+Tree是一種可以自我平衡的資料結構，讓資料維持是排序好的狀態，順序存取(Sequential Access)、插入(insertion)、deletion(刪除)的時間複雜度都是**O(log n)**。

→ 像是允許兩個以上children的二元樹

→ 最適合會做大規模讀取/寫資料的系統

## B+TREE 特性
* M-way search tree(M等於每個節點可以有幾個branch，例如二元樹是2-way)
* 完美平衡(每個leaf node都是一樣的depth)
* 每個不是根(root)的內節點(inner node)都至少比M/2-1大
* 每個鍵值數為k的內節點都有k+1的不為null的子節點

所有不是在最下層的節點都稱做內節點(Inner Node)，最底層的被稱為葉節點(Leaf Node)
葉節點與相鄰的節點會有一個雙向鏈結(Doubly LinkedList)

![](https://i.imgur.com/uo8CuQs.jpg)

葉節點存放資料與其相對應的鍵值(Key)

![](https://i.imgur.com/O2UQ5Ym.jpg)

## B+ Tree Nodes 樹節點
每個B+ Tree裡的節點都存放一個由key/value組所組成的陣列。

→ key一定會是column裡實際的值

→ value取決於是內節點還是葉節點(內節點存放的只是索引，而葉節點會存放Record ID或是實際的tuple資料)

**陣列一定是排序好的!**

## B+ Tree: Leaf Node Values 葉節點裡的值
### 作法#1: Record Ids
→ 指標指向index所對應到的tuple位置
![](https://i.imgur.com/g9dVxFK.jpg)


### 作法#2: Tuple Data
→ tuple的資料直接存在葉節點裡

→ Secondary Indexes have to store the record id as their values
![](https://i.imgur.com/dnZOh3E.jpg)

## B-Tree vs. B+ Tree
### B-Tree
  
將鍵與值存在所有樹裡的節點

→ 更省空間因為每個key只會在tree裡出現一次

### B+ Tree
[B+ Tree視覺化網站](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)

B+ Tree將真正的值存在葉節點裡，內節點不過是幫助搜尋的指引而已。

PageID為相鄰的葉節點的索引
![](https://i.imgur.com/1WWnXEx.jpg)

常見的用法是將鍵存唯一個陣列，然後相對應的值再存一個陣列
![](https://i.imgur.com/skND8uv.jpg)

## B+ Tree: Insert
[看不懂參考這裡](https://www.cnblogs.com/nullzx/p/8729425.html)

1）若為空樹，創建一個葉節點，然後將記錄插入其中，如果葉節點沒有滿則插入操作結束。

2）針對葉子類型節點：根據key值找到葉節點，向這個葉子節點插入記錄。插入後，若當前節點key的個數小於等於m-1，則插入結束。否則將這個葉子節點分裂成左右兩個葉子節點，左葉子節點包含前m/2個記錄，右節點包含剩下的記錄，將第m/2+1個記錄的key進位到父節點中（父節點一定是索引類型節點），進位到父節點的key左孩子指針向左節點,右孩子指針向右節點。將當前節點的指針指向父節點，然後執行第3步。

3）針對索引類型節點：若當前節點key的個數小於等於m-1，則插入結束。否則，將這個索引類型節點分裂成兩個索引節點，左索引節點包含前(m-1)/2個key，右節點包含m-(m-1)/2個key，將第m/2個key進位到父節點中，進位到父節點的key左孩子指向左節點, 進位到父節點的key右孩子指向右節點。將當前節點的指針指向父節點，然後重複第3步。

* 假設我們的B+ Tree Max. Degree為3(每個節點最多只能有3個元素)
  
![](https://i.imgur.com/htq9fk6.jpg)
* 插入4，如果L有足夠空間，就完成!

![](https://i.imgur.com/WvmsMLm.jpg)

* 插入5，沒有空間了，要將L分裂成L及新的葉節點L2
  → 將裡面的元素平均分配，複製中間鍵
  → 分裂後中間key成為索引節點中的key，分裂後當前節點指向了父節點(根節點)
  
![](https://i.imgur.com/g0JNqJL.jpg)


## B+ Tree: Delete
如果葉子節點中沒有相應的key，則刪除失敗。否則執行下面的步驟

1）刪除葉節點中對應的key。刪除後若節點的key的個數大於等於Math.ceil(m-1)/2 – 1，刪除操作結束,否則執行第2步。

2）若兄弟節點key有富餘(大於Math.ceil(m-1)/2 – 1)，向兄弟節點借一個記錄，同時用借到的key替換父節點(指當前節點和兄弟節點共同的父節點)中的key，刪除結束。否則執行第3步。

3）若兄弟節點中沒有富餘的key,則當前節點和兄弟節點合併成一個新的葉子節點，並刪除父節點中的key（父節點中的這個key兩邊的孩子指針就變成了一個指針，正好指向這個新的葉子節點），將當前節點指向父節點（必為索引節點），執行第4步（第4步以後的操作和B樹就完全一樣了，主要是為了更新索引節點）。

4）若索引節點的key的個數大於等於Math.ceil(m-1)/2 – 1，則刪除操作結束。否則執行第5步

5）若兄弟節點有剩餘，父節點key下移，兄弟節點key上移，刪除結束。否則執行第6步

6）當前節點和兄弟節點及父節點下移key合併成一個新的節點。將當前節點指向父節點，重複第4步。

注意，通過B+樹的刪除操作後，索引節點中存在的key，不一定在葉子節點中存在對應的記錄。

下面是一顆5階B樹的刪除過程，5階B+Tree的節點最少2個key，最多4個key。

![](https://i.imgur.com/6jlQXGc.jpg)

* 刪除22，刪除後結果如下圖
![](https://i.imgur.com/LczbKHk.jpg)
因為刪除後key大於2，刪除完成

* 刪除18
![](https://i.imgur.com/1mYVQ0Y.jpg)
刪除後當前節點只有一個key(0017)，而兄弟節點有四個key，可以從兄弟節點借一個0019的key，然後同時更新將父節點中的關鍵自由0019變為0020，刪除結束

![](https://i.imgur.com/8aYcJqa.jpg)

* 刪除7
![](https://i.imgur.com/Sk6ntss.jpg)

當前的key數小於2，左兄弟節點也沒有富餘的key(如果借給當前節點的話自己就不夠了)，所以當前節點和左兄弟節點合併，並刪除父節點中的key，當前節點指向父節點

![](https://i.imgur.com/lHDwtIm.jpg)

此時當前節點的key小於2(只剩0009)，兄弟節點的key也沒有富餘，所以富節點的關鍵字下移，和兩個子節點合併，結果如下:

![](https://i.imgur.com/m62ksCR.jpg)

## B+ Tree設計選擇
在設計B+ Tree架構的時候要考慮到幾個地方:
* 合併門檻(Merge Threshold)
* 重複的索引(Non-Unique Indexes)
* 變動長度的鍵值(Variable Length Keys)
* 前綴壓縮(Prefix Compression)

這些等到要用到時再做筆記.....


### 參考資料
[concurrent B+ tree](https://hackmd.io/@w8qbx0fdRK2ETRnEzcLK2A/SyjKs-mxg?type=view)