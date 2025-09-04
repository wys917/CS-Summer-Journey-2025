# leetcode206-反转链表
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
