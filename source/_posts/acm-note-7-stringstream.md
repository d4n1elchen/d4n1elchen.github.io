---
title: ACM note (7) - stringstream
tags:
  - ACM
id: 1137
categories:
  - Algorithm
date: 2017-02-18 15:59:25
---

stringstream可以將string轉為iostream物件，用iostream的基本輸入輸出操作來操作string，可用於連續的字串轉整數

```cpp
#include<string>
#include<iostream>
#include<sstream>
using namespace std;
int main()
{
  int i;
  string str("87 88 89");
  stringstream ss(str);
  while(ss >> i) cout << i << "+1 = " << i+1 << endl;
}
```

output

```text
87+1 = 88
88+1 = 89
89+1 = 90
```