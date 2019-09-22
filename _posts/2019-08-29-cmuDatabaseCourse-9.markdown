---
layout:     post
title:      "9. 索引並行控制 Index Concurrency Control"
subtitle:   "CMU Database System Note"
date:       2019-08-29
author:     "Ryan"
header-style: text
catalog:    true
tags:
    - Open Course
    - Database
    - CMU
---

之前所討論的資料結構都是假設是單一執行緒的(Single Threaded)，但是我們必須要能夠支援多執行緒(Multi-threaded)才能夠將~~多核CPU榨乾~~，而且多執行緒才可以讓多個交易在同一時間存取同一筆資料。

# 1. 並行控制(Concurrency Control)
要能達到並行控制，DBMS需要使用一種協定(Protocol)或是方法，來確保同一個物件在並行的操作下，得到的結果會是正確的。

