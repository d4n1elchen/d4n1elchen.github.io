---
title: LeetCode Note - Binary Lifting
date: 2020-06-16 04:36:31
tags: [ICPC, algorithm, programming contest, Leetcode]
categories: algorithm
---

[LC 1483. Kth Ancestor of a Tree Node](https://leetcode.com/problems/kth-ancestor-of-a-tree-node/)

## Problem Statement

You are given a tree with n nodes numbered from 0 to n-1 in the form of a parent array where parent[i] is the parent of node i. The root of the tree is node 0.

Implement the function getKthAncestor(int node, int k) to return the k-th ancestor of the given node. If there is no such ancestor, return -1.

The k-th ancestor of a tree node is the k-th node in the path from that node to the root.

> 中文翻譯: 有一顆 n 個 node 的樹，node 以 0 到 n-1 編號，並給定長度為 n 的陣列 parent，parent[i] 為 node i 的父節點 (編號)，node 0 為樹的 root。實作一個 function getKthAncestor(int node, int k)，回傳 node 的第 k 個 ancestor，如果該 ancestor 不存在，回傳 -1。第 k 個 ancestor 定義為由該節點出發往 root 的 path 上第 k 個訪問的節點。

## Example
```
Tree:
     0
   /   \
  1     2
 / \   / \
3   4 5   6

Input:
["TreeAncestor","getKthAncestor","getKthAncestor","getKthAncestor"]
[[7,[-1,0,0,1,1,2,2]],[3,1],[5,2],[6,3]]

Output:
[null,1,0,-1]

Explanation:
TreeAncestor treeAncestor = new TreeAncestor(7, [-1, 0, 0, 1, 1, 2, 2]);

treeAncestor.getKthAncestor(3, 1);  // returns 1 which is the parent of 3
treeAncestor.getKthAncestor(5, 2);  // returns 0 which is the grandparent of 5
treeAncestor.getKthAncestor(6, 3);  // returns -1 because there is no such ancestor
```

> 每組測資第一個操作為輸入 n 以及 parent 陣列，然後接下來會有連續數次的 getKthAncestor 請求。

## Constraints

* 1 <= k <= n <= 5*10^4
* parent[0] == -1 indicating that 0 is the root node.
* 0 <= parent[i] < n for all 0 < i < n
* 0 <= node < n
* There will be at most 5*10^4 queries.

## Thought

最簡單的想法就是紀錄每個節點的 parent 為何，每次 query 就從該節點往回走 k 次，如果在 k 歸零之前走到根節點表示該次 query 失敗，retur -1，否則 return 走道的節點

假設總共的 query 次數是 M 次，則複雜度為 $O(kM)$，k 和 M 最大都是 5e4，相乘會超過 1e9，顯然這會超時

有一個小小的優化是我們可以在最一開始就建立好每個節點查詢第 k 個 ancestor 的查詢表，這樣 query 的複雜度就是 $O(1)$，但建表仍然是 $O(kM)$，依舊會超時

這題的正確解法是使用 Binary Lifting 技巧，我們知道任意整數都可以用 2 的次方相加表示，也就是整數的二進位表達

例如 $5 = {101}\_{bin} = 2^0 + 2^2$，$42 = {101010}\_{bin} = 2^1 + 2^3 + 2^5$

因此我們只需要紀錄每個節點的第 $2^i$ 個 ancestor，每一次不是往前跳一個 parent，而是往前跳 $2^i$ 個 parent，如此總共要跳的次數只剩下 $log(k)$

舉例來說，如果 $k = 42$，總共只需要往前跳三次

$ancestor(node, 42) = ancestor(ancestor(ancestor(node, 2^5), 2^3), 2^1)$

k 值最大為 5e4，因此每個節點我們只需要儲存總共 $log(5 \cdot 10^4) = 16$ 個 ancestor (保險起見待會實作是用 20)

建表的方法如下，因為 $2^i = 2^{i-1} + 2^{i-1}$ 所以 $ancestor(node, 2^i) = ancestor(ancestor(node, 2^{i-1}), 2^{i-1})$ 這就是我們建表的遞迴式了，我們可以用 bottom-up 的方式來建出這個表

查詢時我們可以透過觀察 k 的 binary representation 來知道我們要往前查詢幾個節點，若 k 的第 $i$ 個 bit 為 $1$ 則需要往前跳 $2^i$ 個節點

## Time Complexity

對每一個 node 來說建表的複雜度為 $O(logk)$，整個表一共 $O(nlogk)$

每一次查詢的複雜度為 $O(logk)$，總共 M 次查詢，總體複雜度為 $O(nlogk + mlogk)$

## Full Solution

```cpp
class TreeAncestor {
private:
    vector<vector<int>> p;
public:
    TreeAncestor(int n, vector<int>& parent) {
        p = vector<vector<int>>(n, vector<int>(20, -1));
        for (int i=0; i<n; ++i) {
            p[i][0] = parent[i];
        }
        for (int i=1; i<20; ++i) {
            for (int j=0; j<n; ++j) {
                if (p[j][i-1] == -1) p[j][i] = -1;
                else p[j][i] = p[p[j][i-1]][i-1];
            }
        }
    }
    
    int getKthAncestor(int node, int k) {
        for (int i=0; i<20 && k>>i && node != -1; ++i) {
            if ((k >> i) & 1) node = p[node][i];
        }
        return node;
    }
};

/**
 * Your TreeAncestor object will be instantiated and called as such:
 * TreeAncestor* obj = new TreeAncestor(n, parent);
 * int param_1 = obj->getKthAncestor(node,k);
 */
 ```