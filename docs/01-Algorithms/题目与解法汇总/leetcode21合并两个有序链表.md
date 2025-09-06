# leetcode-21-合并两个有序链表
!!! info "题目"
    将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

    **示例 1：**
    ![alt text](https://assets.leetcode.com/uploads/2020/10/03/merge_ex1.jpg)
    > 输入：l1 = [1,2,4], l2 = [1,3,4]
    > 输出：[1,1,2,3,4,4]

    **示例 2：**
    > 输入：l1 = [], l2 = []
    > 输出：[]

    **示例 3：**
    > 输入：l1 = [], l2 = [0]
    > 输出：[0]

如图所示，题目要求合并这两个有序链表：
??? info_示意图
    ![alt text](image.png)
在做这道题目的时候，我希望能不开辟新的内存来存储结果，而是直接使用现有的链表的其中之一，但是当我实际在写代码的时候发现这会带来非常多的临界条件需要额外进行判断，所以我又开始想会不会还是新建一个链表比较合适呢。
就在这时，我突然想到我可以建一个`dummy`节点，这样就能完美的规避额外的条件判断，于是我写出了如下代码：
```c
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2) {
    struct ListNode dummy;
    struct ListNode* tail = &dummy; // tail 就够了，不需要 res, head, cur...

    while (list1 != NULL && list2 != NULL) {
        if (list1->val <= list2->val) {
            tail->next = list1;
            list1 = list1->next;
        } else {
            tail->next = list2;
            list2 = list2->next;
        }
        tail = tail->next;
    }

    // 一行解决所有剩余问题
    tail->next = list1 != NULL ? list1 : list2;

    return dummy.next;
}
```
这道题虽然简单，但却让我真正掌握和理解了`dummy`节点的巧妙之处，我觉得还是很有价值的。