---
layout:     post
title:      "在Win10安裝PostgreSQL遇到安裝問題解決方法"
subtitle:   安裝PostgreSQL遇到"problem running post-install step the database cluster initialization failed"
date:       2019-08-29
author:     "Ryan"
header-style: text
catalog:    true
tags:
    - Database
    - Postgres
    - Technical Solution
---

# 問題
今天在家裡的Win10安裝PostgresSQL 11.1，快安裝完成的時候給我跳"problem running post-install step the database cluster initialization failed"的錯誤，導致Postgres的服務無法啟用。

嘗試過各種方法:
* 創建Postgres用戶 - 沒用
* 用系統管理員打開安裝檔 - 沒用
* locale選C - 沒用

# 解決方法

1.安裝之後出現錯誤沒關係，到Postgres資料夾按右鍵點選內容 -> 安全性 -> 編輯
2.點選User那邊，權限那邊的完全控制要打勾允許

![](https://i.imgur.com/J7lLCek.jpg)

3.在PostgresSQL/bin路徑底下用系統管理員權限打開CMD，輸入:

```
initdb.exe -D D:\PostgresSQL\data -E UTF8
```
中間的路徑是你定義Postgres數據庫的路徑，這一步完成後會在這路徑下加入一堆默認庫文件。

4.接下來啟動Postgres的服務，輸入:

```
pg_ctl -D D:\PostgresSQL\data -l logfile.txt start
```
如果想重啟，後面start改為restart，要關閉則用stop

5.最後建立用戶，輸入以下指令進入PostgreSQL命令行:

```
psql postgres
```
然後執行:
```
CREATE USER Ryan with superuser password '123456';
```
做完以上動作，應該就可以登錄PgAdmin玩資料庫了!