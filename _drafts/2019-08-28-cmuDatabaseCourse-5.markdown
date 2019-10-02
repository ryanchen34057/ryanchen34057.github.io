---
layout:     post
title: 5. 資料庫儲存 II  
categories: [Database]
description: some word here
keywords: 資料庫, Database, Open Course, CMU, SQL
---

[參考來源](https://www.youtube.com/watch?v=NXRgIsH83xE&list=PLSE8ODhjZXja3hgmuwhf89qboV1kOxMx7&index=4)

# 1. 資料表示(Data Representation)

* INTEGER / BIGINT / SMALLINT / TINYINT

→ C/C++ Representation
* FLOAT / REAL vs. NUMERIC / DECIMAL

→ IEEE-754 Standard / Fixed-point Decimals
* VARCHAR / VARBINARY / TEXT / BLOB

→ Header with length, followed by data bytes
* TIME / DATE / TIMESTAMP

→ 32 / 64-bit integer of (micro)seconds since

## Variable Precision Numbers
Variable-precision number型別使用原生C/C++的型別。直接存為IEEE-754定義的型別。
通常會比任意精度(Arbitrary precision)的型別效能要好
例: **FLOAT**, **REAL/DOUBLE**

### 準度錯誤 (Rounding Error)
```c
#include <stdio.h>
int main(int argc, char* argv[]) {
    float x = 0.1;
    float y = 0.2;
    printf("x+y = %.20f\n", x+y);
    printf("0.3 = %.20f\n", 0.3);
}
```
```
output:
x+y = 0.30000001192092895508
0.3 = 0.29999999999999998890
```
## Fixed Precision Numbers
任意精度(Arbitrary precision)的數字型別，在不希望失去數字準確度時使用。

例: **NUMERIC, DECIMAL**

或是儲存為可變化長度的BINARY。
有點像是不是字串的VARCHAR

### 如何處理比分頁還大的Tuple?
大部分的DBMS不會讓Tuple超過一個分頁的大小(Size)

如果要存下比分頁更大的值，DBMS會另外一個叫做Overflow Storage page，用指標指向外部的分頁，也就是**外部儲存**!

→ Postgres: TOAST (>2KB)

→ MySQL: Overflow (> 1/2 size of page)

#### 外部儲存(External Value Storage)
一些系統允許把很大的值存在外部的檔案裡
當作**BLOB**來處理

→ Oracle: **BFILE**

→ MySQL: **FILESTREAM**

因為是外部檔案，DBMS無法操控檔案的內容!

## 表格資料(Information Schema)
>列出資料庫中所有的表格
* SQL-92
```sql
SELECT *
FROM INFORMATION_SCHEMA.TABLES
WHERE table_catalog = '<db name>';
```
* Posgres
```
/d;
```
* MySQL
```
SHOW TABLES;
```
* SQLite
```
.tables;
```

# 2. 資料處理(Workload)
Wiki範例資料庫

![](https://i.imgur.com/ZgIpNun.png)

## OLTP
>On-line Transaction Processing

OLTP是傳統的關係型資料庫的主要應用，主要是基本的、日常的事務處理，例如銀行交易。
通常只會跟資料庫裡的一個實體(Entity)有關連，以下的SQL都算是OLTP:
```sql
SELECT P.*, R.*
FROM pages AS P
INNER JOIN revisions AS R
ON P.latest = R.revID
WHERE P.pageID = ?
```
```sql
UPDATE useracct
SET lastLogin = NOW(),
hostname = ?
WHERE userID = ?
```
```sql
INSERT INTO revisions
VALUES (?,?…,?)
```
## OLAP
>On-line Analytical Processing 

OLAP是資料倉儲系統的主要應用，支援複雜的分析操作，並且提供直觀易懂的查詢結果。
通常會是比較複雜的query，關連到複數的資料庫裡的Entity。
會從OLTP收集來的資料作OLAP的運算，以下為OLAP的SQL範例:
```sql
SELECT COUNT(U.lastLogin),
    EXTRACT(month FROM
        U.lastLogin) AS month
FROM useracct AS U
WHERE U.hostname LIKE '%.gov'
GROUP BY
EXTRACT(month FROM U.lastLogin)
```
### 操作複雜度的關係圖
可以發現OLTP是比較單純的指令，用於寫(write)比較多，而OLAP則是較複雜的指令，用於讀(read)比較多。
![](https://i.imgur.com/rt7h6qJ.png)

# 3. 儲存結構(Storage Model)
## N-ARY Storage Model (NSM)
DBMS將每個屬性的Tuple以排列式儲存在分頁裡。

對OLTP的處理方式最友善，因為OLTP比較常在單一個表格上做操作，並且有很大的"寫"的比例。
### OLTP Query
只要將想找的分頁抓出來，然後藉由Index找出相對應的Tuple，再回傳整個Tuple
![](https://i.imgur.com/zc6ydkm.png)

### OLAP Query
每一個分頁都要掃過(Sequential Scan)，然後比對WHERE敘述句裡的condition(非常慢)
![](https://i.imgur.com/CjIsj8m.png)

#### 好處(Advantage):
* INSERT, UPDATE, DELETE很快速
* 適合需要整個Tuple的Query

####  壞處(Disadvantage):
* 不適合需要瀏覽所有表格裡的資料的Query

## Decomposition Storage Model (DSM)
DBMS將所有Tuple的屬性有順序地存在分頁裡。

→ 又叫做 " Column Store"

→ 對OLAP的資料處理比較適合，因為OLAP只做讀取，並且會做大量瀏覽表格裡所有的屬性。

![](https://i.imgur.com/RIrIQV7.png)


## 要如何重構Tuple? (How to reconstruct the tuple?)
### 選擇1: Fixed-length Offsets
  
→ 讓屬性裡的每個值都一樣長，然後藉由offset去存取 

### 選擇2: Embedded Tuple Ids
  
→ 在每個column裡的值都會有一個tuple id(貌似沒人用?)

#### 好處:
* 減少I/O次數，因為DBMS只需要讀取它需要的資料
* 更好的query處理及資料壓縮
  
#### 壞處:
* 不適合做INSERT, UPDATE, DELETE等query，因為DSM會把tuple分解又湊合起來