---
title: ICPC note (3) - string 建構子
tags: [ICPC, algorithm, programming contest, c++]
id: 1099
categories: c++
date: 2017-02-18 14:21:47
---

# String

這篇紀錄一些常用的STL字串操作，因為太多了所以分成幾篇列舉

## Constructor

```cpp]
string s0("Initial string");         // Initial string
string s1;                           // (empty string)
string s2(s0);                       // Initial string
string s3(s0, 8, 3);                 // str
string s4("1234567890", 5);          // 12345
string s5(10, 'x');                  // xxxxxxxxxx
string s6(s0.begin(), s0.begin()+7); // Initial
```

### assign

assign可以為字串物件賦值，用法與constructor類似

```cpp]
string str;
str.assign(...);
```

### insert

inset在字串中指定位置插入字串，用法也與constructor相似

```cpp]
string str("Already have something.");
str.insert(pos, ...);
str.insert(iterator, ...);
```

### replace

replace取代指定範圍，用法與insert相似

```cpp]
string str("Already have something.");
str.insert(pos, len, ...);
str.insert(start_iter, end_iter, ...);
```
