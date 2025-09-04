# leetcode141题 环形链表

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

