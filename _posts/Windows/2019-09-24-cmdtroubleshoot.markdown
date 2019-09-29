---
layout: post
title: 在命令提示字元(CMD)打指令之後出現：『不是內部或外部命令、可執行的程式或批次檔。』解決方法
categories: [Windows]
description: some word here
keywords: Windows, CMD, 『不是內部或外部命令、可執行的程式或批次檔。』
---

## 問題
今天在建置SQL Server的時候在CMD想查詢SQL Server服務的狀態，但是在CMD打了`net`之後卻出現:

>『不是內部或外部命令、可執行的程式或批次檔。』

## 解決方式
Windows -> 搜尋「環境變數」 -> 選擇系統變數 -> Path -> 新增 

-> 輸入變數值: 『%SystemRoot%\system32』

![](https://i.imgur.com/asoDbOW.png=100x20)


新增變數之後就可以用`net`囉!