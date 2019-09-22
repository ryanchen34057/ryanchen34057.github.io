---
layout:     post
title:      "C# Async / Await用法介紹"
subtitle:   ""
date:       2019-09-11
author:     "Ryan"
header-style: text
tags:
    - C#
    - Asynchronous Programming
---

## 前言
大家都知道程式是由上至下，有順序地逐步執行的。

但是有時候還是會希望它可以不要這麼乖，有些地方如果是要等一下下才會完成的工作，會希望程式不要卡死在那等它完成，先繼續往下跑，這時候就需要 **『異步執行』** (Asynchronous Execution)的幫助!

最常見的用途在使用者介面(GUI)，我們不希望使用者在送出請求後整個程式就完全停擺了! 所以必須要將等候資料或是計算這一段擺在背景執行，以免影響使用者體驗。


而C#提供了`async`及`await`的關鍵字及`Task`，讓我們可以在不用寫很程式碼的情況下撰寫出異步執行的程式碼。





