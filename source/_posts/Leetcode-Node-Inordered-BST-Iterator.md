---
title: Leetcode Note - Inordered BST Iterator
date: 2019-09-25 23:40:34
tags: [ICPC, Algorithm, Programming Contest, Leetcode, BST]
categories: Algorithm
---

[LC 653. Two Sum IV - Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/)

## Problem Statement
> Given a Binary Search Tree and a target number, return true if there exist two elements in the BST such that their sum is equal to the given target.

## Examples
```
Input:
    5
   / \
  3   6
 / \   \
2   4   7

Target = 9

Output: True
```
```
Input:
    5
   / \
  3   6
 / \   \
2   4   7

Target = 28

Output: False
```

## Thought

The strategy is the same as finding two elements in a sorted array that sum of them equals to k. But we cannot perform random access on a BST. The key point of this question is to either rebuild the sorted array by inorder traversal through BST or iteratively go through the BST also in inorder order.

There's a clear and efficient solution posted in the discussion that he implemented a BST Iterator to simulate the two pointer sweeping in the sorted array version.

I try to implement one with some modification to make it more clear and readable.

## Time Complexity

The traverse operation cost O(1) and the loop stop when all the elements are visited once. So overall complexity is O(N).

## Full Solution
```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class BSTIterator {
    TreeNode *curr;
    TreeNode *next;
    stack<TreeNode*> st;
    bool forward;
public:
    BSTIterator(TreeNode *root, bool forward) : curr(root), next(root), forward(forward) {
        while(next) {
            curr = next;
            st.push(curr);
            next = forward? curr->left: curr->right;
        }
    }
    int operator*() { return curr->val; }
    void operator++(int) {
        while(next) {
            curr = next;
            st.push(curr);
            next = forward? curr->left: curr->right;
        }
        curr = st.top(); st.pop();
        next = forward? curr->right: curr->left;
    }
};
class Solution {
public:
    bool findTarget(TreeNode* root, int k) {
        if(!root) return false;
        BSTIterator l(root, true), r(root, false);
        while(*l < *r) {
            if(*l + *r == k) return true;
            else if(*l + *r > k) r++;
            else l++;
        }
        return false;
    }
};
```
