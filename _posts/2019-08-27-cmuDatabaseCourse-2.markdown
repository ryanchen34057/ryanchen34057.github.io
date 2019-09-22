---
layout:     post
title:      "2. 基本SQL語法"
subtitle:   "CMU Database System Note"
date:       2019-08-27
author:     "Ryan"
header-style: text
catalog:    true
tags:
    - Open Course
    - Database
    - CMU
---
# 1. 關聯式語言(SQL)介紹
關聯式語言其實是由下列3種語言所組合而成的
* Data Manipulation Language (DML)

    讓我們可以對資料做存取及操控的語言，主要做插入(INSERT)、更新(UPDATE)、刪除(DELETE)、取回(RETRIEVE)等操作。
* Data Definition Language (DDL)
    
    讓我們可以對資料庫做操作，像是CREATE、ALTER、DELETE等。
* Data Control Language (DCL)

    讓我們可以管理對資料庫的存取權。例如Grant、Revoke等存取權的控制。
    
# 2. 資料型態(Data Type)
首先，先來看一下關聯式資料庫裡都可以存些什麼資料型態吧!
## 整數(Integer)

整數又可以依照整數資料的大小需求，選擇一個夠用但又不會浪費空間的整數形態，能表示的數越多代表需要的儲存空間越大。

|型態|Byte(s)|最小值(Signed/Unsigned)|最大值(Signed/Unsigned)|
|:--:|:--:|:--:|:--:|
|TINYINT(長度)|1|-128/ 0|127/ 255|
|SMALLINT(長度)|2|-32768/ 0|32767/ 65535|
|MEDIUMINT(長度)|3|-8388608/ 0|8388607/ 16777215|
|INT(長度)|4|-2147483648/ 0|214748647/ 4294967295|
|BIGINT(長度)|8|-9223372036854775808/ 0|9223372036854775807/ 18446744073709551614|

## 浮點數(Float)

「FLOAT」和「DOUBLE」型態的欄位可以用來儲存包含小數的數值，它們是一種佔用儲存空間比較小，執行運算比較快的型態。不過因為它們是使用「近似值」來儲存你的數值，所以如果你需要儲存完全精準的數值，就不能使用這兩種型態。

另外一種可以儲存小數數值的「DECIMAL」型態就可以用來儲存完全精準的數值，儲存在這個型態中的數值，不論是查詢或是運算，都不會有任何誤差，不過「DECIMAL」型態佔用的儲存空間就比「FLOAT」和「DOUBLE」型態大。「DECIMAL」型態在MySQL還有一個一樣的關鍵字是「NUMERIC」，這兩種型態完全一樣。

|型態|Byte(s)|最大長度|說明|
|:--:|:--:|:--:|:--:|
|FLOAT[(長度,小數位數)]|4|255, 30|單精確度浮點數(近似值)|
|DOUBLE[(長度,小數位數)]|8|255, 30|雙精確度浮點數(近似值)|
|DECIMAL[(長度[,小數位數])]|10, 0|65, 30|自行指定位數的精確值|

## 字串(String)
字串型態有下列幾種：
|型態|最大長度|儲存空間|說明|
|:--:|:--:|:--:|:--:|
|CHAR[(長度)]|255|指定的長度|固定長度的字串，預設長度為1|
|VARCHAR(長度)|65535|字元個數加1或2bytes|變動長度的字串|
|TINYTEXT|255|字元個數加1byte|變動長度的字串|
|TEXT|65535|字元個數加2bytes|變動長度的字串|
|MEDIUMTEXT|16,772,215|字元個數加3bytes|變動長度的字串|
|LONGTEXT|4,294,967,295|字元個數加4bytes|變動長度的字串|

固定長度與變動長度的兩種字串型態都可以儲存字串，差異在儲存的文字個數小於型態指定的長度時，變動長度實際儲存的空間會小一些，以下列的
在SQL中可以使用LENGTH函式來查詢每個字元儲存時所使用的空間:
```SQL
SELECT s, LENGTH(s) FROM table
```

另外，字串還可以設定Collation，collation會影響字串排列順序。
|欄位名稱|型態|字元集|Collation|
|:--:|:--:|:--:|:--:|
|string1|VARCHAR(6)|latin1|latin1_general_ci|
|string2|VARCHAR(6)|latin1|latin1_general_cs|
上列表格中欄位的字元集都指定為「latin1」，不過「string1」欄位的collation設定為「latin1_general_ci」，代表"case insensitive"，表示排序時不區分大小寫；「string2」欄位設定為「latin1_general_cs」，代表"case sensitive"，表示排序時會區分大小寫。

## 二進制(Binary)
  
二進位制的字串型態是使用位元組(byte)為單位來儲存字串資料，跟非二進位制的字串類似，它也提供許多應用在不同長度的型態：

|型態|最大長度(byte)|儲存空間|說明|
|:--:|:--:|:--:|:--:|
|BINARY[(長度)]|255|指定的長度|固定長度的字串，預設長度為1|
|VARBINARY(長度)|65535|長度加1或2bytes|變動長度的字串|
|TINYBLOB|255|byte個數加1byte|變動長度的字串|
|BLOB|65535|byte個數加2bytes|變動長度的字串|
|MEDIUMBLOB|16,772,215|byte個數加3bytes|變動長度的字串|
|LONGBLOB|4,294,967,295|byte個數加4bytes|變動長度的字串|

二進制的儲存方式跟字串很類似，BINARY儲存固定長度的bytes，VARBINARY儲存變動長度的bytes。正常來說用VARBINARY會比BINARY還要節省空間。
二進制的資料不可以指定字元集或是Collation，但是可以使用它們來儲存任何語言的文字，也可以儲存音樂或圖片資料。
  
## 日期與時間(Date and Time)
下列幾個可以儲存日期與時間資料的欄位型態：
|型態|儲存空間|範圍|說明|
|:--:|:--:|:--:|:--:|
|DATE|3|日期|1000-01-01 ~ 9999-12-31|
|TIME|3|時間|-838:59:59 ~ 838-59-59|
|DATETIME|8|日期與時間|1000-01-01 00:00:00 ~ 9999-12-31 23:59:59|
|YEAR[(4/2)]|1|西元年|1901 ~ 2155[YEAR(4)] 1970 ~ 2069[YEAR(2)]|
|TIMESTAMP|4|日期與時間|1970-01-01 00:00:00 ~ 2037-12-31 23:59:59|

因為日期中的西元年份可以使用4個或2個數字，使用2個數字的時候，「70」到「99」表示「1970」到「1999」；如果是「00」到「69」就是「2000」到「2069」。

時間(TIME)型態可以儲存時、分、秒的資料，範圍從「-838:59:59」到「838:59:59」。
這個儲存時間資料的範圍可能會跟你想的不太一樣。一般來說，時間資料指的是從「00:00:00」到「23:59:59」，也就是一天的時間。MySQL的時間型態欄位可以讓你儲存類似「經過的時間」這樣的資料：
```sql
INSERT INTO dttable(t) VALUES ('200:30:00'); /*200 HOURS AND 30 MINS*/
INSERT INTO dttable(t) VALUES ('-1:20:30'); /*1 HOURS AND 20 MINS AND 30 SEC BEFORE*/
```
在指定一個時間資料的時候，你可以省略秒或分，省略的部份，MySQL都會幫你設定為「0」:
```SQL
INSERT INTO dttable (t) VALUES ('200'); /*200:00:00*/
```

日期與時間(DATETIME)型態可以儲存完整的年、月、日與時、分、秒資料，範圍從「1000-01-01 00:00:00」到「9999-12-31 23:59:59」。在表示一個日期與時間資料的時候，日期與時間之間，至少要使用一個空白隔開。時間部份的時、分、秒都可以省略，省略的部份，MySQL都會幫你設定為「0」：
```SQL
INSERT INTO dttable (dt) VALUES ('2000-01-01 10:10:10');
INSERT INTO dttable (dt) VALUES ('2000-01-01 200:10:10'); /*WRONG: 時間格式要符合範圍!!*/
```

「TIMESTAMP」型態的格式與「DATETIME」一樣，都包含完整的年、月、日與時、分、秒資料，不過它使用的儲存空間只有4bytes，是「DATETIME」型態的一半。

「TIMESTAMP」也是MySQL日期與時間型態中具有「時區」特性的型態。它可以儲存從「1970-01-01 00:00:00」到目前經過的秒數。這個起始日期與時間使用「Coordinated Universal Time、UTC」世界標準時間為儲存資料的依據，它與「Greenwich Mean Time、GMT」格林威治標準時間是一樣的。

全世界分為許多不同時區(time zone)，所有時區都使用跟標準時間的差異來當作自己的標準時間。以台灣來說，你會在安裝Windows平台的電腦中，經由控制台裡的日期和時間，看到這個關於時區的設定。

# 3. Data Definition Language(DDL)
DDL就是做跟表(Table)有關的命令集。

## 創建(CREATE)
```SQL
Create Table
CREATE TABLE student (
		student_id INT PRIMARY KEY,
        name VARCHAR(20),
        major VARCHAR(20)
);
```

### 創建有foreign key的table
```SQL
CREATE TABLE branch(
	branch_id INT PRIMARY KEY,
    branch_name VARCHAR(40),
    mgr_id INT,
    mgr_start_date DATE,
    FOREIGN KEY(mgr_id) REFERENCES employee(emp_id) ON DELETE SET NULL
);
```
#### ON DELETE 的參數
在設定foreign key時，ON DELETE的參數會影響當被參照的值被刪除時，table裡的值要做什麼變化。在MySQL中有4種可以選擇:
* CASCADE - 會將有所關聯的紀錄行也會進行刪除或修改。
* SET NULL - 會將有所關聯的紀錄行設定成 NULL。
* NO ACTION - 有存在的關聯紀錄行時，會禁止父資料表的刪除或修改動作。
* RESTRICT - 與 NO ACTION 相同。

### Describe table
列出table的詳細資訊
```sql=
DESCRIBE student;
```
### Delete table
刪除Table
```sql
DROP TABLE student;
```

## 更動(ALTER)
### 新增屬性(Add Attribute)
```sql
/*Add a attribute called gpa*/
ALTER TABLE student ADD gpa DECIMAL(3, 2);

ALTER TABLE employee ADD FOREIGN KEY(branch_id) 
REFERENCES branch(branch_id) ON DELETE SET NULL;
```

### 刪除列(Drop column)
```sql
/*Drop a column called gpa*/
ALTER TABLE student DROP gpa;
```

# 4. Data Manipulation Language(DML)
## 插入資料(INSERT)
```sql
INSERT INTO student VALUES(1, 'Jack', 'Biology');

/*Insert but leaving certain field null*/
INSERT INTO student(student_id, name) VALUES(3, 'Claire');
```
## 更新資料(UPDATE)
```sql
UPDATE student SET major = 'Bio' WHERE major = 'Biology';

/*Add or*/
UPDATE student SET major = 'Biochemistry' 
WHERE major = 'Bio' OR major = 'Chemistry';

/*Affect all rows*/
UPDATE student SET major = 'Biochemistry'
```
## 刪除資料(DELETE)
```sql
DELETE FROM student WHERE student_id = 5;
```
## 選擇資料(SELECT)
```sql
/*Select all*/
SELECT * FROM student;

/*Select specific fields*/
SELECT name, major FROM student;

/*With order (ascending)*/
SELECT student.name, student.major FROM student ORDER BY name ASC;

/*With order (descending)*/
SELECT student.name, student.major FROM student ORDER BY name DESC;

/*Multiple order by*/
SELECT * FROM student ORDER BY major, student_id DESC;

/*AS (給選取的屬性自訂的名稱)*/
SELECT first_name AS forename, last_name AS surname FROM employee

/*DISTINCT (只列出相異的屬性)*/
SELECT DISTINCT sex FROM employee;

/*LIMIT*/
SELECT * FROM student LIMIT 2;

/*增加條件式*/
SELECT name, major FROM student WHERE major = 'biology';

/*檢查是否列出的值有包含在括弧內*/
SELECT * FROM student WHERE name IN ('Claire', 'Kate', 'Mike');

/*計算row的數量，其實括弧內輸入什麼屬性都一樣*/
SELECT COUNT(emp_id) FROM employee;
/*計算列出值的平均值*/
SELECT AVG(salary) FROM employee WHERE sex = 'M';

/*計算列出值的和*/
SELECT SUM(salary) FROM employee;
```