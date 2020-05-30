---
title: ICPC note (4) - string 基本操作
tags: [ICPC, algorithm, programming contest, c++]
id: 1122
categories: c++
date: 2017-02-18 15:43:29
---

## append

### +=

```cpp
string name ("John");
string family ("Smith");
name += " K. ";         // c-string
name += family;         // string
name += '\n';           // character
```

### append

```cpp
string name ("John");
string family ("Smith");
name.append(" K. ");    // c-string
name.append(family);    // string
name.append('\n');      // character
```

### push_back

```cpp
string line ("This is a line.");
line.push_back('\n');   // character
```

## erase

### erase

```cpp
str.erase(pos, len);
```

### pop_back

```cpp
str.pop_back();
```

## find

### find string

```cpp
str.find(str2);  // return iterator
str.rfind(str2); // find last
```

### find charactor

```cpp
str.find_first_of(char); // return iterator
str.find_last_of(char);  // find last
```

## substr

```cpp
str.substr(pos, len);
str.substr(pos);      // from pos to end
```
