---
title: ACM note (2) - set & disjoint set
tags:
  - ACM
id: 1049
categories:
  - Algorithm
date: 2017-02-18 14:05:59
---

# Set

set即集合，特性為集合中的元素不會重複

STL中的set有點像只有key的map，在新增元素的時候會進行排列，若新增重複的元素則忽略，但結構跟行為其實差蠻多的

```cpp
// Member functions:
// insert, erase, clear, find
#include <iostream>
#include <set>
using namespace std;
set<int> s;
int main ()
{
  s.insert(2);
  s.insert(3);
  s.insert(6);
  s.insert(1);
  s.insert(2);

  for(set<int>::iterator it=s.begin(); it!=s.end(); it++)
    cout << *it << endl;
  return 0;
}
```

output

```text
1
2
3
6
```

# Disjoint set

互斥集合，即多個元素不互相重複的集合，例如班上的同學分組做報告，一個人只能在一個組，不過一組的人數可以不固定，分出來的組即為disjoint set

Disjoint set有三個操作，Union, Find, Split，分別為將兩個disjoint set合成一個、找尋某個元素位於哪個set、將一個set拆成兩個

Disjoint set可以用一般的陣列實做，將所有元素用正整數編號，並紀錄每個人所在的set，紀錄的方式為指定一個人作為該set的組長，組長的編號就是該set的名稱，初始化時將每個人都自己一組，自己為該組組長，union時將某一個組的組長指令給另外一組，find時遞迴找到set名稱和自己的編號一樣時即為組長

```cpp
#include<iostream>
using namespace std;

const int n = 10;
int root[n];

void Init()
{
  for(int i=0; i<n; i++) root[i] = i;
}

int Find(int x)
{
  if(x==root[x]) return x;
  return root[x] = Find(root[x]);
}

int Union(int x, int y)
{
  if(Find(x) != Find(y))
    root[Find(y)] = Find(x);
}

int main()
{
  Init();
  Union(2, 3);
  Union(2, 4);
  Union(2, 5);
  Union(6, 7);
  Union(6, 8);
  Union(6, 9);

  int now = root[0];
  printf("{%d", 0);
  for(int i=1; i<n; i++) {
    if(root[i] != now) {
      now = root[i];
      printf("} {%d", i);
    } else {
      printf(",%d", i);
    }
  }
  printf("}\n");
  return 0;
}
```

output

```text
{0} {1} {2,3,4,5} {6,7,8,9}
```