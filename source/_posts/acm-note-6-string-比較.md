---
title: ACM note (6) - string 比較
tags: [ACM]
id: 1128
categories: Algorithm
date: 2017-02-18 15:45:21
---

## compare

### ==, !=, >, <, >=, <=

STL字串可使用比較運算子
等於和不等於為比較字串內容是否相等
大於小於為判斷最後一個匹配相等的字元位置大於或小於另一個字串

```cpp
string str1;
string str2;
str1 = "apple";
str2 = "apple";
if (str1==str2) cout << "str1 and str2 are equal\n";
str1 = "red apple";
str2 = "green apple";
if (str1!=str2) cout << "str1 and str2 are not equal\n";
str1 = "apple";
str2 = "applepie";
if (str1< str2) cout << "str1 is less than str2\n";
str1 = "applepie is good";
str2 = "applepie";
if (str1> str2) cout << "str1 is greater than str2\n";
```

### compare

比較運算子其實是根據compare的結果來判斷的，compare共有三個結果:
- 0 兩個字串相等
- \>0 去除相等部份後前者大於後者的長度
- 其實簡單來說就是去除相等的部份後兩者長度相減

```cpp
string str1;
string str2;
str1 = "apple";
str2 = "apple";
cout << str1.compare(str2) << '\n';
str1 = "red apple";
str2 = "green apple";
cout << str1.compare(str2) << '\n';
str1 = "apple";
str2 = "applepie";
cout << str1.compare(str2) << '\n';
str1 = "applepie is good";
str2 = "applepie";
cout << str1.compare(str2) << '\n';
```

output

```text
0
11
-3
8
```