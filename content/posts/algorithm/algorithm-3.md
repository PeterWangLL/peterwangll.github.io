---
title: 后序遍历的魅力
date: 2024-10-08
categories:
  - Algorithm
nolastmod: true
cover: /img/pic-3.jpg
draft: true
---
两道题都是递归做后面的事情，再来处理当下的事情，很值得思考。
### 树中两个结点的最低公共祖先
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q)
        return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null)
        return root;
    return left == null ? right : left;
}
```
### 从尾到头打印链表(递归实现)
```java
public List<Integer> printListReversingly(ListNode head) {
    if (head == null)
        return new ArrayList<>();
    List<Integer> res = printListReversingly(head.next);
    res.add(head.val);
    return res;
}
```
