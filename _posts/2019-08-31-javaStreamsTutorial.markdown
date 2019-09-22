---
layout:     post
title:      "Java 8 Streams用法與範例"
subtitle:   ""
date:       2019-08-31
author:     "Ryan"
header-style: text
tags:
    - Java
    - Functional Programming
---
[參考來源](https://www.youtube.com/watch?v=t1-YZ6bF-g0)
## Streams是啥?
Streams從Java 8開始支援，新的Java 8藉由Streams把函數式編程(Functional Programming)帶入了Java編程中。(什麼是[函數式編程](https://zh.wikipedia.org/zh-tw/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)?)

## Stream有甚麼好處?
* 讓Java編程更有效率
* 使用大量的lambda expression(匿名函數)
* ParallelStreams的api讓多執行緒的操作更加容易了

## Streams的組成
![](https://i.imgur.com/p8Bre1U.jpg)
* Stream來源(Stream Source)
  Streams的來源可以是Collections, Lists, Sets, ints, longs, doubles, arrays, lines of file

* Stream的操作可以是中間操作(intermediate operation)或是終端操作(terminal operation)
  - **中間操作 (intermediate operation)**: 例如filter、map、sort等函數會回傳stream，所以可以把它們串連起來操作。
  - **終端操作 (terminal operation)**: 像是forEach、collect、reduce等函數會回傳void或是其他不是stream的結果，所以操作到這邊就會中斷。

## 中間操作 (intermediate operation)
可以容許0個以上的中介操作，排列順序會影響執行的順序，例如如果在比較大的資料上面做操作的話，**先filter再sort或map效能會比較好一點**(減少後面函數要處理的資料量)

較大的資料量也可以用ParallelStream來使用多執行緒來操作。

中間操作有:
```java
anyMatch()
distinct()
filter()
findFirst()
flatmap()
map()
skip()
sorted()
```

## 終端操作 (terminal operation)
**只能有一個終端操作!**
* **forEach**對所有元素執行同一個方法
* **collect**將所有元素放入一個集合(Collection)裡
* 其他方法只回傳一個元素，像是:

```java
count()
max()
min()
reduce()
summayStatistics()
```

## 範例 Examples
### 1. 印出整數列 Integer Stream
```java
IntStream
    .range(1, 10)
    .forEach(System.out::print); // java裡的方法引用! 類名::方法名
System.out.println();
```
Output:
```
123456789
```

### 2. 印出整數列，跳過前幾個元素

```java
IntStream
    .range(1, 10)
    .skip(5) // 跳過頭5個元素
    .forEach(x -> System.out.println(x)); // 匿名函數
System.out.println();
```
Output:
```
6
7
8
9
```

### 3. 印出整數列的和

```java
System.out.println(
    IntStream
    .range(1, 5)
    .sum());
System.out.println();
)
```
Output:
```
10
```

### 4. Stream.of用法
```java
Stream.of("Ava", "Aneri", "Alberto") // 將一連串的資料變成collection的stream
    .sorted() // 排序
    .findFirst() // 抓出第一個資料
    .ifPresent(System.out::println); //如果有資料的話印出來
```
Output:
```
Alberto
```

### 5. 過濾陣列裡的資料
> 找出所有S開頭的名字
```java
String[] names = {"Al", "Ankit", "Kushal", "Brent", "Sarika", "amanda", "Hans", "Shivika"};
Arrays.stream(names) // same as Stream.of(names)
    .filter(x -> x.startsWith("S"))
    .sorted()
    .forEach(System.out::println());
```
Output:
```
Sarika
Shivika
```

### 6. map()用法
> 計算整數陣列所有元素平方的平均

```java
Arrays.stream(new int[] {2, 4, 6, 8, 10})
    .map(x -> x * x)
    .average() // 求出平均
    .ifPresent(System.out::println);
```
Output:
```
44.0 // 印出double而不是int!!
```

## 7. 結合map()及filter()
```java
List<String> people = Arrays.asList("Al", "Ankit", "Kushal", "Brent", "Sarika", "amanda", "Hans", "Shivika");
people
    .stream()
    .map(String::toLowerCase)
    .filter(x -> x.startsWith("a"))
    .forEach(System.out::println);
```
Output:
```
al
ankit
amanda
```

## 8. 文字檔處理
```java
Stream<String> bands = Files.lines(Paths.get("bands.txt"));
bands
    .sorted()
    .filter(x -> x.length() > 13);
    .forEach(System.out::println);
bands.close();
```

## 9. reduce()用法
> 將陣列裡所有數相加

```java
double total = Stream.of(7.3, 1.5, 4.8);
    .reduce(0.0, (Double a, Double b) -> a + b); // 0.0是起始點
System.out.println("Total = " + total);
```

## 10. summaryStatistics()用法
```java
IntSummaryStatistics summary = IntStream.of(7, 2, 19, 88, 73, 4, 10)
    .summaryStatistics();
System.out.println(summary);
```
Output:
```
IntSummaryStatistics{count=7, sum=203, min=2, average=29.000000, max=88}
```