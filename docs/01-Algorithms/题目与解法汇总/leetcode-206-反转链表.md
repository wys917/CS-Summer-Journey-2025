# LeetCode 206: 反转链表
!!! info "题目"
    给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

    **示例 1：**
    ![alt text](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)
    > 输入：head = [1,2,3,4,5]
    > 输出：[5,4,3,2,1]

    **示例 2：**
    ![alt text](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)
    > 输入：head = [1,2]
    > 输出：[2,1]

    **示例 3：**
    > 输入：head = []
    > 输出：[]

本题较为简单，所以直接给出代码，仅作记录用
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverseList(struct ListNode* head) {
    if (head==NULL){
        return head;
    }
    struct ListNode* tail=head->next;
    struct ListNode* H=head;
    struct ListNode* cur=head;
    while(tail!=NULL){
        head=tail;
        tail=tail->next;
        head->next=cur;
        cur=head;
    }
    H->next=NULL;
    return head;
}
```
