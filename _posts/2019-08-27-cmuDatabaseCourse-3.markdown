---
layout:     post
title:      "3. 進階SQL語法"
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
[參考](https://www.youtube.com/watch?v=80atcA6gBU8&list=PLSE8ODhjZXja3hgmuwhf89qboV1kOxMx7&index=2)


### 本文的範例所用的資料庫: 
分別為學生資料、課程資料及學生選課資料
#### student
```
+--------+------------+------+------+-------+
| name   | login      | age  | gpa  | sid   |
+--------+------------+------+------+-------+
| shakur | sharkur@cs |   26 |  3.5 | 53655 |
| Kanye  | kayne@cs   |   39 |    4 | 53666 |
| Bieber | jbieber@cs |   22 |  3.9 | 53688 |
+--------+------------+------+------+-------+
```
#### course
```
+--------+------------------------------+
| cid    | name                         |
+--------+------------------------------+
| 15-445 | Database Systems             |
| 15-721 | Advanced Database Systems    |
| 15-823 | Advanced Topics in Databases |
| 15-826 | Data Mining                  |
+--------+------------------------------+
```
#### enrolled
```
+-------+--------+-------+
| sid   | cid    | grade |
+-------+--------+-------+
| 53666 | 15-445 | C     |
| 53688 | 15-721 | A     |
| 53688 | 15-826 | B     |
| 53655 | 15-445 | B     |
| 53666 | 15-721 | C     |
+-------+--------+-------+
```


# 1. 聚合函數(Aggregate Function)
聚合函數讓我們可以對SELECT所選取的值做進一步的操作。
可做的操作大概有以下幾種:

* AVG(col) → 回傳平均值 
* MIN(col) → 回傳最小值
* MAX(col) → 回傳最大值 
* SUM(col) → 回傳所有值的和
* COUNT(col) → 回傳某個列的所有數量
  
<p style="color:red;">※ 聚合函數只能用在SELECT輸出的列表中!!</p>

>題目: Get # of students with a “@cs” login:

```sql
/*1*/
SELECT COUNT(login) AS cnt
FROM student WHERE login LIKE '%@cs'
/*2*/
SELECT COUNT(*) AS cnt
FROM student WHERE login LIKE '%@cs'
/*3*/
SELECT COUNT(1) AS cnt
FROM student WHERE login LIKE '%@cs'
/*每個output都一樣!*/
```

## 聚合函數指令(Aggregation Command)
### 複數個Aggregate

>Get the number of students and their average GPA that have a “@cs” login.

```sql
SELECT AVG(gpa), COUNT(sid)
FROM student WHERE login LIKE '%@cs'
```
```
output: 
+-------------------+------------+
| AVG(gpa)          | COUNT(sid) |
+-------------------+------------+
| 3.800000031789144 |          3 |
+-------------------+------------+
```
### Distinct Aggregate
```sql
SELECT COUNT(DISTINCT login)
FROM student WHERE login LIKE '%@cs'
```
```
output: 
+-----------------------+
| COUNT(DISTINCT login) |
+-----------------------+
|                     3 |
+-----------------------+
```
>Get the average GPA of students enrolled in each course.

```sql
SELECT AVG(s.gpa), e.cid
FROM student AS s, enrolled AS e WHERE s.sid = e.sid; // WRONG!
```
```
output:
+--------------------+--------+
| AVG(s.gpa)         | cid    |
+--------------------+--------+
| 3.8600000381469726 | 15-445 | // random cid!
+--------------------+--------+
```
### GROUP BY
搭配聚合函數使用，用來將查詢結果中特定欄位值相同的資料分為若干個群組，而每一個群組都會傳回一個資料列。
<p style="color:red;">※在SELECT裡的非聚合值(Non-aggregated values)必須要出現在GROUP BY敘述句裡!!!</p>

```sql
SELECT AVG(s.gpa), e.cid
FROM student AS s, enrolled AS e WHERE s.sid = e.sid GROUP BY e.cid;
```
```
output:
+--------------------+--------+
| AVG(s.gpa)         | cid    |
+--------------------+--------+
|               3.75 | 15-445 |
|  3.950000047683716 | 15-721 |
| 3.9000000953674316 | 15-826 | // right!!
+--------------------+--------+
```
### HAVING
就像是給GROUP BY敘述句用的WHERE filter一樣
```sql
SELECT AVG(s.gpa) AS avg_gpa, e.cid
FROM student AS s, enrolled AS e 
WHERE s.sid = e.sid 
    AND avg_gpa > 3.9 /*doesn't work!!! aggregate的函數還沒算出來!*/
GROUP BY e.cid;
```
```sql
SELECT AVG(s.gpa) AS avg_gpa, e.cid
FROM student AS s, enrolled AS e 
WHERE s.sid = e.sid
GROUP BY e.cid
HAVING avg_gpa > 3.9; /*算出來了!!*/
```
```
output:
+--------------------+--------+
| avg_gpa            | cid    |
+--------------------+--------+
|  3.950000047683716 | 15-721 |
| 3.9000000953674316 | 15-826 |
+--------------------+--------+
```
# 2. 字串處理(String Operation)
![](https://i.imgur.com/wfFcseU.png)


## LIKE
用來做string matching
### 字串運算子(String match operators)
* "%" - 表示任意0個或多個字元。可匹配任意類型和長度的字元，有些情況下若是中文，可使用兩個百分號（%%）表示。
```sql
SELECT * FROM enrolled AS e
WHERE e.cid LIKE '15-%';
```
* "_" - 表示任意單個字元。匹配單個任一字元，它常用來限制運算式的字元長度語句
```sql
SELECT * FROM student AS s
WHERE s.login LIKE '%@c_';
```
### 字串連接(String Concatenation)
**MySQL: CONCAT**
```sql
SELECT name FROM student
WHERE login = CONCAT(LOWER(name), '@cs');
```
**SQL-92: ||**
```sql
SELECT name FROM student
WHERE login = LOWER(name) || '@cs';
```
**MSSQL: +**
```sql
SELECT name FROM student
WHERE login = LOWER(name) + '@cs'
```
# 3. 輸出重導向(Output Redirection)
### 將儲存Query的結果到其他的Table。

需注意:
* Table不能是已經被定義的。
* Table會有跟input一樣數量的Column及一樣的型別。

**MySQL**
```sql
CREATE TABLE CourseIds (
SELECT DISTINCT cid FROM enrolled);
```
**SQL - 92**
```sql
SELECT DISTINCT cid INTO CourseIds
FROM enrolled;
```
### 把query結果的tuple插入其他Table

括號裡的SELECT一定要有跟目標Table一樣的Column
每個DBMS對於duplicates的處理上，語法不盡相同
* SQL - 92
```sql
INSERT INTO CourseIds
(SELECT DISTINCT cid FROM enrolled);
```

# 4. 輸出控制(Output Control)
## 排序
### ORDER BY [ASC|DESC]
```sql
SELECT sid, grade FROM enrolled
WHERE cid = '15-721'
```
```
output:
+-------+-------+
| sid   | grade |
+-------+-------+
| 53688 | A     |
| 53688 | B     |
| 53655 | B     |
| 53666 | C     |
| 53666 | C     |
+-------+-------+
```
```sql
SELECT sid FROM enrolled
WHERE cid = '15-721'
ORDER BY grade DESC, sid ASC
```
```
output:
+-------+
| sid   |
+-------+
| 53666 |
| 53688 |
+-------+
```
### LIMIT
限制回傳的tuple數量，可以利用offset來做到"回傳範圍"的限制。

```sql
SELECT sid, name FROM student
WHERE login LIKE '%@cs'
LIMIT 10;
```
```sql
SELECT sid, name FROM student
WHERE login LIKE '%@cs'
LIMIT 20 OFFSET 10;
```
# 5. 巢狀查詢(Nested Queries)
Query裡有另一個Query。通常比較難去做最佳化。

```sql
SELECT name FROM student 
WHERE sid /*outer query*/
    IN (SELECT sid FROM enrolled /*inner query*/);
```
>Get the names of students in '15-445'

```sql
SELECT name FROM student 
WHERE sid 
    IN (SELECT sid FROM enrolled WHERE cid = '15-445');
```
也可以倒過來寫:
```sql
SELECT (
    SELECT S.name FROM student AS S
    WHERE S.sid = E.sid
) AS sname
FROM enrolled AS E 
WHERE cid = '15-445';
```
## ALL
必須滿足所有sub-query裡的條件

>Find student record with the highest id that is enrolled in at least one 
course.

```sql
SELECT sid, name FROM student
WHERE sid => ALL(
    SELECT sid FROM enrolled
)
```
```sql
SELECT sid, name FROM student 
WHERE sid IN(
    SELECT MAX(sid) FROM enrolled
);
```
```sql
SELECT sid, name FROM student 
WHERE sid IN(
    SELECT sid FROM enrolled ORDER BY sid DESC LIMIT 1
);
```
## ANY 
必須滿足sub-query裡其一的條件

```sql
SELECT name FROM student
WHERE sid = ANY(
    SELECT sid FROM enrolled
    WHERE cid = '15-445'
);
```
## IN
與ANY相同

## EXISTS
至少會傳一個row

>Find all courses that has no students enrolled in it.

```sql
SELECT cid, name FROM course WHERE NOT EXISTS(
    SELECT * FROM enrolled 
    WHERE course.cid = enrolled.cid
);
```
```
output:
+--------+------------------------------+
| cid    | name                         |
+--------+------------------------------+
| 15-823 | Advanced Topics in Databases |
+--------+------------------------------+
```
# 6. 視窗函數(Window Function)
在組像是統計型報表所要用的資料集時，往往會連結許多次資料表、使用複雜的語法來組出想要的資料集 為了兼顧到取出資料的效能跟成果，需要有足夠的經驗跟邏輯才能達到使用較少運算資源快速取得資料 而透過SQL Server提供的視窗函數，大量的減少語法的使用，並且讓之後維護的人員能夠快速了解語法 而產出的過程中也能有一定品質的效率
特色
對與某個row有相關聯的一組tuple執行計算
有點像aggregation但是tuple不會被合成在一起

## 函數
* ROW_NUMBER() → 目前ROW的號碼
* RANK() → 目前ROW的順序位置
* OVER(PARTITION BY *) → 執行視窗函數時，OVER關鍵字定義出將tuple集結起來的方式
```sql
SELECT *, ROW_NUMBER() OVER() AS row_num FROM enrolled;
```
```
output:
+-------+--------+-------+---------+
| sid   | cid    | grade | row_num |
+-------+--------+-------+---------+
| 53666 | 15-445 | C     |       1 |
| 53688 | 15-721 | A     |       2 |
| 53688 | 15-826 | B     |       3 |
| 53655 | 15-445 | B     |       4 |
| 53666 | 15-721 | C     |       5 |
+-------+--------+-------+---------+
```
```sql
SELECT cid, sid,
ROW_NUMBER() OVER (PARTITION BY cid)
FROM enrolled
ORDER BY cid;
```
```sql
output:
+--------+-------+-------------------------------------+
| cid    | sid   | ROW_NUMBER() OVER(PARTITION BY cid) |
+--------+-------+-------------------------------------+
| 15-445 | 53666 |                                   1 |
| 15-445 | 53655 |                                   2 |
| 15-721 | 53688 |                                   1 |
| 15-721 | 53666 |                                   2 |
| 15-826 | 53688 |                                   1 |
+--------+-------+-------------------------------------+
```

>Find the student with the highest grade for each course.

// only supported in postgres!
```sql
SELECT * FROM (
    SELECT *,
    RANK() OVER (PARTITION BY cid
    ORDER BY grade ASC)
    AS rank
    FROM enrolled) AS ranking
WHERE ranking.rank = 1
```
# 7. 常見的Table敘述(Common Table Expression)
## WITH

```sql
WITH cteName AS (
    SELECT 1
)
SELECT * FROM cteName;
```
```
output:
+---+
| 1 |
+---+
| 1 |
+---+
```
```sql
WITH cteName (col1, col2) AS (
    SELECT 1, 2
)
SELECT col1 + col2 FROM cteName;
```
```
output:
+-------------+
| col1 + col2 |
+-------------+
|           3 |
+-------------+
```
>Find student record with the highest id that is enrolled in at least one course.
```SQL
WITH cteSource(maxId) AS(
    SELECT MAX(sid) FROM enrolled)
SELECT sid, name FROM student WHERE student.sid = cteSource.maxId;
```

# 8. 聯集/合併查詢結果(UNION)
## employee
```
+------------+-----------+--------+
| first_name | last_name | emp_id |
+------------+-----------+--------+
| David      | Chen      |    100 |
| Michael    | Josh      |    102 |
| Josh       | Wang      |    106 |
| Sherry     | Lin       |    108 |
| Sara       | Chou      |    110 |
+------------+-----------+--------+
```

## branch
```
+-----------+----------+--------+
| branch_id | name     | mgr_id |
+-----------+----------+--------+
|       200 | Neihu    |    102 |
|       300 | Shindien |    106 |
|       400 | Banqiao  |    100 |
+-----------+----------+--------+
```

在關聯式資料庫中常常會需要合併多個查詢結果，而合併(UNION)指的是將「多個」查詢敘述所得到的結果合成為「一個」。
下列的範例就是將employee的查詢結果跟branch的查詢結果用UNION結合起來:

>找出所有員工的姓及所有分店的名稱

```sql
SELECT first_name FROM employee UNION SELECT name FROM branch;
```
output:
```
+----------+
| name     |
+----------+
| David    |
| Michael  |
| Josh     |
| Sherry   |
| Sara     |
| Neihu    |
| Shindien |
| Banqiao  |
+----------+
```

# 9. 結合查詢(JOIN、INNER/OUTER JOIN、LEFT/RIGHT OUTER JOIN)
UNION是把多個查詢結果合併起來，而**JOIN則是可以把多個Table結合起來一起查詢**。
JOIN又依照不同的需求可以分成:
* [INNER] JOIN
* OUTER JOIN
* LEFT JOIN
* RIGHT JOIN
  
## 內部結合 [INNER] JOIN (INNER可以省略掉)
如果要找出所有分店的店經理要怎麼下Query呢?
用**INNER JOIN** 可以把兩個表格結合起來查詢，以下的範例是將employee跟branch兩個表格的查詢結果結合起來，然後加上條件查詢。
```sql
SELECT employee.emp_id, employee.first_name, branch.branch_name 
FROM employee 
JOIN branch ON employee.emp_id = branch.mgr_id;
```
output:
```
+------------+-----------+----------+
| first_name | last_name | name     |
+------------+-----------+----------+
| Michael    | Josh      | Neihu    |
| Josh       | Wang      | Shindien |
| David      | Chen      | Banqiao  |
+------------+-----------+----------+
```
條件式除了可以用ON以外，還有另外一種選擇，就是用USING。
但是這種寫法的前提是**兩種欄位名稱要一樣**，所以上述的範例就不行。
```sql
SELECT employee.emp_id, employee.first_name, branch.branch_name
FROM employee
JOIN branch USING (emp_id) /*前提是employee跟branch裡都有叫做emp_id的欄位*/
```

## 外部結合 LEFT/RIGHT[OUTER] JOIN
假設某部門的資料表格長這樣:
```sql
/*empno=員工編號, ename=員工名, deptno=部門編號*/
+-------+-------+--------+
| empno | ename | deptno |
+-------+-------+--------+
|  7369 | SMITH |     20 |
|  7469 | ALISA |   NULL |
|  7499 | ALLEN |     30 |
|  7899 | RYAN  |   NULL |
|  8000 | PETER |     50 |
+-------+-------+--------+
```
然後部門資料表格長這樣:
```
+--------+------------+
| deptno | dname      |
+--------+------------+
|     20 | ACCOUNTING |
|     30 | RESEARCH   |
|     50 | SALES      |
|     60 | IT         |
|     70 | OPERATIONS |
+--------+------------+
```
如果使用INNTER JOIN的方式結合兩個表格:
```sql
SELECT emp.empno, emp.ename, emp.deptno, dept.dname 
FROM emp 
JOIN dept ON emp.deptno = dept.deptno;
```
output:
```
+-------+-------+--------+------------+
| empno | ename | deptno | dname      |
+-------+-------+--------+------------+
|  7369 | SMITH |     20 | ACCOUNTING |
|  7499 | ALLEN |     30 | RESEARCH   |
|  8000 | PETER |     50 | SALES      |
+-------+-------+--------+------------+
```
會發現沒有部門編號的Ryan跟Alisa沒有在查詢結果裡面!!

這是因為INNER JOIN的查詢一定要符合ON後面結合條件的資料才會出現!

如果想查詢的資料是「包含部門名稱的員工資料，沒有分派部門的員工也要出現」，那你就要使用「OUTER JOIN」，這種結合查詢通常稱為「外部結合」。
外部結合又分為兩種:
* LEFT JOIN
* RIGHT JOIN

### LEFT JOIN
如果你想要查詢的資料是「包含部門名稱的員工資料，沒有分派部門的員工也要出現」，也就是希望不符合結合條件的資料也要出現的話，就要換成使用「LEFT OUTER JOIN」來執行結合查詢。

```sql
SELECT emp.empno, emp.ename, emp.deptno, dept.dname 
FROM emp 
LEFT JOIN dept ON emp.deptno = dept.deptno;
```
output:
```
+-------+-------+--------+------------+
| empno | ename | deptno | dname      |
+-------+-------+--------+------------+
|  7369 | SMITH |     20 | ACCOUNTING |
|  7469 | ALISA |   NULL | NULL       |
|  7499 | ALLEN |     30 | RESEARCH   |
|  7899 | RYAN  |   NULL | NULL       |
|  8000 | PETER |     50 | SALES      |
+-------+-------+--------+------------+
```

### RIGHT JOIN
如果想查詢的資料是「包含部門名稱的員工資料，沒有員工的部門也要出現」，就要換成「RIGHT OUTER JOIN」來執行結合查詢。
```sql
SELECT emp.empno, emp.ename, emp.deptno, dept.dname 
FROM emp 
RIGHT JOIN dept ON emp.deptno = dept.deptno;
```
output:
```
+-------+-------+--------+------------+
| empno | ename | deptno | dname      |
+-------+-------+--------+------------+
|  7369 | SMITH |     20 | ACCOUNTING |
|  7499 | ALLEN |     30 | RESEARCH   |
|  8000 | PETER |     50 | SALES      |
|  NULL | NULL  |   NULL | IT         |
|  NULL | NULL  |   NULL | OPERATIONS |
+-------+-------+--------+------------+
```