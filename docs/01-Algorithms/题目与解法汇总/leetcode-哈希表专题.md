# LeetCode 专题: 哈希表

!!! info "专题说明"
    本页面汇总了与哈希表（Hash Table）相关的 LeetCode 经典题目，旨在通过一系列问题，深入理解哈希表的原理、应用场景及解题技巧。

    **核心思想：**
    哈希表利用键值对（Key-Value）的映射关系，实现 O(1) 时间复杂度的快速查找、插入和删除。它是解决算法问题，特别是涉及查找、计数、去重等场景的利器。


### LeetCode 1: 两数之和

!!! info "题目"
    给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** `target`  的那 **两个** 整数，并返回它们的数组下标。

    你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

    你可以按任意顺序返回答案。

    **示例 1：**
    > 输入：nums = [2,7,11,15], target = 9
    > 输出：[0,1]
    > 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。

    **示例 2：**
    > 输入：nums = [3,2,4], target = 6
    > 输出：[1,2]

    **示例 3：**
    > 输入：nums = [3,3], target = 6
    > 输出：[0,1]

---

传统解法的时间复杂度为`O(n^2)`，效率低下，主要原因在于需要低效的遍历每一个可能的值，这就导致如果想要找的两个数都位于数组的结尾时需要遍历很多无效的数据。使用哈希表则非常方便，每次遍历到的数都会存入表中，哈希值相同的数据则会通过链表的形式存储，哈希值查找的时间复杂度为`O(1)`,链表寻找的时间复杂度为`O(n)`，所以总的时间复杂度为`O(n)`。另外在具体写代码的时候有一点需要重点注意，那就是每一次更新哈希表的时候，新加入的node的地址需要在函数内永久保存，所以要通过`malloc`在堆上建立而不是在栈上定义。

`node *snode=(node *)malloc(sizeof(node));`而不是`node snode`。

下面是我写的最终版代码：

```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
 
int hash(int numsSize,int value){
    int index=value%numsSize;
    if(index<0){
        index+=numsSize;
    }
    return index;
}
int* twoSum(int* nums, int numsSize, int target, int* returnSize) {
    int *result=(int *)malloc(2*sizeof(int));
    typedef struct node{
        int value;
        int key;
        struct node *next;
    } node;
    node **list=(node **)malloc(sizeof(node *)*numsSize);
    for(int i=0;i<numsSize;i++){
        list[i]=NULL;
    }
    for(int i=0;i<numsSize;i++){
        int complement=target-nums[i];
        int index=hash(numsSize,complement);
        node *current=list[index];
        while(current!=NULL){
            if(current->value==complement){
                result[0]=i;
                result[1]=current->key;
                *returnSize=2;
                return result;
            }
            current=current->next;
        }
        int index_i=hash(numsSize,nums[i]);
        node *snode=(node *)malloc(sizeof(node));
        snode->value=nums[i];
        snode->key=i;
        snode->next=list[index_i];
        list[index_i]=snode;
    }
    return result;
}
```

**总结：传统的查找方式为遍历，时间复杂度为`O(n^2)`，而哈希表查找更加高效，时间复杂度为`O(n)`。**

### LeetCode 217: 存在重复元素

!!! info "题目"
    给你一个整数数组 `nums` 。如果任一值在数组中出现 **至少两次** ，返回 `true` ；如果数组中每个元素互不相同，返回 `false` 。

    **示例 1：**
    > 输入：nums = [1,2,3,1]
    > 输出：true

    **示例 2：**
    > 输入：nums = [1,2,3,4]
    > 输出：false

    **示例 3：**
    > 输入：nums = [1,1,1,3,3,4,3,2,4,2]
    > 输出：true

---

与此相似的还有第217题，比较类似所以不单独写感悟，下面直接给出本人的代码：

```C
int hash(int numsSize,int value){
    int index;
    index=value%numsSize;
    if(index<0){
        index+=numsSize;
    }
    return index;
}
bool containsDuplicate(int* nums, int numsSize) {
    typedef struct node{
        int value;
        struct node *next;
    }node;
    node **hashtable=(node **)malloc(sizeof(node *)*numsSize);
    for(int i=0;i<numsSize;i++){
        hashtable[i]=NULL;
    }
    for(int i=0;i<numsSize;i++){
        int index=hash(numsSize,nums[i]);
        node *current=hashtable[index];
        while(current!=NULL){
            if(current->value==nums[i]){
                return true;
            }
            current=current->next;
        }
        node *newnode=(node *)malloc(sizeof(node));
        newnode->value=nums[i];
        newnode->next=hashtable[index];
        hashtable[index]=newnode;

    }
    return false;


}
```

### LeetCode 242: 有效的字母异位词

!!! info "题目"
    给定两个字符串 `s` 和 `t` ，编写一个函数来判断 `t` 是否是 `s` 的字母异位词。

    **注意：** 若 `s` 和 `t` 中每个字符出现的次数都相同，则称 `s` 和 `t` 互为字母异位词。

    **示例 1:**
    > 输入: s = "anagram", t = "nagaram"
    > 输出: true

    **示例 2:**
    > 输入: s = "rat", t = "car"
    > 输出: false



```c
int hash(char s,int size){
    int idx=s-'a';
    idx%=size;
    return idx;
}
bool isAnagram(char* s, char* t) {
    int size=26;
    typedef struct node{
        char val;
        struct node* next;
    }node;
    node **hashtable=(node **)malloc(sizeof(node *)*size);
    for(int i=0;i<size;i++){
        hashtable[i]=NULL;
    }
    int len=strlen(s);
    for(int i=0;i<len;i++){
        int idx=hash(s[i],size);
        node *newnode=(node *)malloc(sizeof(node));
        newnode->val=s[i];
        newnode->next=hashtable[idx];
        hashtable[idx]=newnode;
    }
    int len_t=strlen(t);
    if(len!=len_t){
        return false;
    }
    for(int j=0;j<len_t;j++){
        int idx=hash(t[j],size);
        if(hashtable[idx]==NULL){
            return false;
        }else{
            hashtable[idx]=hashtable[idx]->next;
        }

    }
    return true;
}
```

---





