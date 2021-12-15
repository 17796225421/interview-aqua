#### [92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

难度中等1101

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置 `left` 到位置 `right` 的链表节点，返回 **反转后的链表** 。

 

**示例 1：**

![img](image/rev2ex2.jpg)

```
输入：head = [1,2,3,4,5], left = 2, right = 4
输出：[1,4,3,2,5]
```

**示例 2：**

```
输入：head = [5], left = 1, right = 1
输出：[5]
```

```c
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        if(left==right)return head;
        ListNode*dummy=new ListNode(0,head);
        ListNode*first=dummy,*cur=head->next;
        for(int i=1;i<=left-1;i++){
            first=first->next;
            cur=cur->next;
        }
        ListNode *pre=first->next;
        for(int i=left+1;i<=right;i++){
            ListNode*prepre=first->next,*nex=cur->next;
            first->next=cur;
            pre->next=nex;
            cur->next=prepre;
            cur=nex;
        }
        return dummy->next;
    }
};
```

