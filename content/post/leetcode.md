---
title: "Leetcode题解"
date: 2020-06-24T15:11:50+08:00
draft: false
tags: 
- code
categories: 
- 面试经验
---

### Leetcode

## 142. 环形链表II

https://leetcode-cn.com/problems/linked-list-cycle-ii/

### 题目描述

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

### 代码

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *fast=head, *slow=head;
        while(true){
            if(fast == nullptr || fast->next == nullptr)return nullptr;
            slow = slow -> next;
            fast = fast->next->next;
            if(fast == slow) break;
        }
        fast = head;
        while(fast != slow){
            slow = slow->next;
            fast = fast->next;
        }
        return fast;
    }
};
```

## 452. 用最少数量的箭引爆气球

https://leetcode-cn.com/problems/minimum-number-of-arrows-to-burst-balloons/

### 题目描述

在二维空间中有许多球形的气球。对于每个气球，提供的输入是水平方向上，气球直径的开始和结束坐标。由于它是水平的，所以y坐标并不重要，因此只要知道开始和结束的x坐标就足够了。开始坐标总是小于结束坐标。平面内最多存在104个气球。

一支弓箭可以沿着x轴从不同点完全垂直地射出。在坐标x处射出一支箭，若有一个气球的直径的开始和结束坐标为 xstart，xend， 且满足  xstart ≤ x ≤ xend，则该气球会被引爆。可以射出的弓箭的数量没有限制。 弓箭一旦被射出之后，可以无限地前进。我们想找到使得所有气球全部被引爆，所需的弓箭的最小数量。

### 代码

``` cpp
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        sort(points.begin(),points.end(),[](const vector<int> &o1, const vector<int> &o2) {
      return (o1[1] < o2[1]);
    });
        int ans = 0;
        int pre = 0; 
        for(auto it = points.begin();it!=points.end();it++){
            if(ans == 0){
                ans++;
                pre = (*it)[1];
            }else{
                if(pre < (*it)[0]){
                    ans++;
                    pre = (*it)[1];
                }
            }
        }
        return ans;
    }
};
```

## 236. 二叉树的最近公共祖先

https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/

### 题目描述

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]

### 代码

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
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == nullptr)return nullptr;
        if(root == p || root == q)return root;
        TreeNode *lson = lowestCommonAncestor(root->left,p,q);
        TreeNode *rson = lowestCommonAncestor(root->right,p,q);
        if(lson == nullptr)return rson;
        if(rson == nullptr)return lson;
        if(lson && rson)return root;
        return nullptr;
    }
};
```

