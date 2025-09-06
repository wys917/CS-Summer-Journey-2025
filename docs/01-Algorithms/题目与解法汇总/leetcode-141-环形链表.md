# LeetCode 141: 环形链表

!!! info "题目"
    给你一个链表的头节点 `head` ，判断链表中是否有环。

    如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**如果 `pos` 是 `-1`，则在该链表中没有环**。注意：`pos` 不作为参数进行传递，仅仅是为了标识链表的实际情况。

    **不允许修改** 链表。

    **示例 1：**
    ![alt text](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)
    > 输入：head = [3,2,0,-4], pos = 1
    > 输出：true
    > 解释：链表中有一个环，其尾部连接到第二个节点。

    **示例 2：**
    ![alt text](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist_test2.png)
    > 输入：head = [1,2], pos = 0
    > 输出：true
    > 解释：链表中有一个环，其尾部连接到第一个节点。

    **示例 3：**
    ![alt text](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist_test3.png)
    > 输入：head = [1], pos = -1
    > 输出：false
    > 解释：链表中没有环。

这题要求我们设计一个算法，判断给定的链表内部是否存在环。解决问题的关键就是设计一个合适的判断链表中存在环的方式，那么这里直接给出一个合理的方案：快慢指针，每次循环时，慢指针向前走一步，快指针则走两步，如此一来二者之间的差距每一次都会增加1，所以如果存在环的话他们一定会相撞，即`slow==fast`。

下面是我写的最终的代码：

```c
typedef struct ListNode Node;
bool hasCycle(struct ListNode *head) {
   if(head==NULL||head->next==NULL){
    return false;
   }
   Node *slow=head;
   Node *fast=head->next;
   while(slow->next!=NULL&&fast->next!=NULL&&fast->next->next!=NULL){
        slow=slow->next;
        fast=fast->next->next;
        if(slow==fast){
            return true;

        }
   }
   return false;
}
```

