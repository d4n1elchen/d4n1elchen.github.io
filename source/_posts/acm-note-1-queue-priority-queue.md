---
title: ACM note (1) queue & priority queue
tags: [ACM]
id: 1022
categories: Algorithm
date: 2017-02-18 13:21:12
---

# Queue

Queue即先進先出(FIFO list)，常用於排隊問題

STL內建之Queue:

```cpp
// Member functions:
// push, pop, front, back, empty, size
#include<queue>
#include<cstdio>
using namespace std;
queue<int> q;
int main() {
    q.push(2);
    q.push(3);
    q.push(1);
    while(!q.empty()) {
        printf("%d\n", q.front());
        q.pop();
    }
}

```

output

```text
2
3
1
```

# Priority Queue

Priority Queue確保Queue裡面的元素為降序排列

STL內建的Priority Queue:

```cpp
// Member functions:
// push, pop, top, empty, size
#include<cstdio>
#include<queue>
using namespace std;
priority_queue<int> prio;
int main() {
    prio.push(2);
    prio.push(3);
    prio.push(1);
    while(!prio.empty()) {
        printf("%d\n", prio.top());
        prio.pop();
    }
}

```

output

```text
3
2
1
```