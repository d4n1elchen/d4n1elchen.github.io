---
title: ICPC note (5) - string I/O
tags: [ICPC, Algorithm, Programming Contest]
id: 1125
categories: Algorithm
date: 2017-02-18 15:44:42
---

## cin

```cpp
string str;
cin >> str;
```

## getline

getline一次讀取一行，換行字元不會被存入，讀取成功時回傳cin物件，eof時回傳eofbit

```cpp
string str;
getline(cin, str);
```

## iterator &amp; cout

```cpp
string str ("Test string");
for ( string::iterator it=str.begin(); it!=str.end(); ++it)
  cout << *it;
cout << '\n';
cout << str << '\n';
```
