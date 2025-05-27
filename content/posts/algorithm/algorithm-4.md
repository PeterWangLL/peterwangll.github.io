---
title: 排序链表
date: 2024-10-21
categories:
  - Algorithm
nolastmod: true
cover: /img/pic-4.jpg
---
链接：https://leetcode.cn/problems/7WHec2/solutions/2852644/lcr-077-pai-xu-lian-biao-by-stormsunshin-769f/
归并排序思想，自顶向下实现
* 时间复杂度：O(nlogn)
* 空间复杂度：O(logn)
```java
class Solution {
    public ListNode sortList(ListNode head) {
        // 递归终止条件：链表长度为1时它是有序的
        if (head == null || head.next == null)
            return head;
        // 将链表一分为二
        ListNode slow = getMidPoint(head);
        ListNode head2 = slow.next;
        slow.next = null;
        // 得到有序的两条链表
        ListNode list1 = sortList(head);
        ListNode list2 = sortList(head2);
        // 合并有序链表
        return mergeTwoList(list1, list2);
    }
    // 取链表中点，双指针遍历
    public ListNode getMidPoint(ListNode head) {
        ListNode slow = head;
        ListNode fast = head.next;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }
    // 合并有序链表
    public ListNode mergeTwoList(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(-1);
        ListNode p = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                p.next = list1;
                list1 = list1.next;
            } else {
                p.next = list2;
                list2 = list2.next;
            }
            p = p.next;
        }
        if (list1 != null) {
            p.next = list1;
        }
        if (list2 != null) {
            p.next = list2;
        }
        return dummy.next;
    }
}
```