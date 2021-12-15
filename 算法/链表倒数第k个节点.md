#### [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

难度简单301

输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

例如，一个链表有 `6` 个节点，从头节点开始，它们的值依次是 `1、2、3、4、5、6`。这个链表的倒数第 `3` 个节点是值为 `4` 的节点。

 

**示例：**

```c
给定一个链表: 1->2->3->4->5, 和 k = 2.

返回链表 4->5.
```

方法二：双指针
思路与算法

快慢指针的思想。我们将第一个指针 \textit{fast}fast 指向链表的第 k + 1k+1 个节点，第二个指针 \textit{slow}slow 指向链表的第一个节点，此时指针 \textit{fast}fast 与 \textit{slow}slow 二者之间刚好间隔 kk 个节点。此时两个指针同步向后走，当第一个指针 \textit{fast}fast 走到链表的尾部空节点时，则此时 \textit{slow}slow 指针刚好指向链表的倒数第kk个节点。

我们首先将 \textit{fast}fast 指向链表的头节点，然后向后走 kk 步，则此时 \textit{fast}fast 指针刚好指向链表的第 k + 1k+1 个节点。

我们首先将 \textit{slow}slow 指向链表的头节点，同时 \textit{slow}slow 与 \textit{fast}fast 同步向后走，当 \textit{fast}fast 指针指向链表的尾部空节点时，则此时返回 \textit{slow}slow 所指向的节点即可。

代码

```c
class Solution {
public:
    ListNode* getKthFromEnd(ListNode* head, int k) {
        ListNode* fast = head;
        ListNode* slow = head;

        while (fast && k > 0) {
            fast = fast->next;
            k--;
        }
        while (fast) {
            fast = fast->next;
            slow = slow->next;
        }

        return slow;
    }
};

```

复杂度分析

时间复杂度：O(n)O(n)，其中 nn 为链表的长度。我们使用快慢指针，只需要一次遍历即可，复杂度为 O(n)O(n)。

空间复杂度：O(1)O(1)。



