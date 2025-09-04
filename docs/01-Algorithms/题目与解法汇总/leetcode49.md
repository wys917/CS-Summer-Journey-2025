# leetcode第49题



#### **第一步：最初的想法与必然的崩溃**



我的解题思路起点是清晰且正确的：识别异位词的关键，是为它们找到一个唯一的“身份证”。最直观的方法就是**排序**，将“eat”和“tea”都变成“aet”。然后，我需要一个**哈希表**来根据这个“身份证”进行分组。

于是，我写下了第一版充满天真幻想的代码。在我的想象中，我只需要 `malloc` 一个 `Node` 结构体，然后就可以像变魔术一样，随意地往 `Node->key` 和 `Node->class.val` 这些指针里塞东西。



```C
// 我最初的、必然会崩溃的代码片段
Node *NewNode = (Node *)malloc(sizeof(Node));
strcpy(NewNode->key, sorted_string);     // 灾难：NewNode->key 是一个野指针
NewNode->class.val[0] = original_string; // 灾难：NewNode->class.val 也是野指针
```

**结果**：现实用一个冷冰冰的**段错误 (Segmentation Fault)** 给了我一记响亮的耳光。程序告诉我：你只盖了个空房子（`Node`结构体），就想让客人（数据）住进去？你连卧室（`key`）和客厅（`val`）都没建！

**暴露的短板**：这一步，暴露了我对C语言指针最致命的无知：**结构体里的指针，也需要单独、额外地为它们分配内存。** 这是我碰上的第一堵墙。



#### **第二步：所有权的混乱——偷来的身份证 vs 复印的身份证**



在被AI的无情嘲讽点醒后，我学会了要为指针成员分配内存。但为了省事，我走到了一个新的十字路口，并毫不犹豫地选择了错误的方向：我直接让我的哈希表指向了输入的 `strs[i]`。



```C
// 犯下更隐蔽错误的代码片段
NewNode->class.val[0] = strs[i];
```

这引出了我们之间关于“内存所有权”的地狱级讨论。我一开始固执地认为，既然输入数据不变，直接引用它既省事又省内存。但我终于明白了，这就像偷了别人的身份证放进自己的钱包，一旦别人报警挂失，我手里的就是一张废纸。我创建的数据结构，其生命周期可能比输入数据更长，直接引用外部指针会埋下“悬垂指针”的巨大隐患。

**思路的转变**：我最终认识到，`malloc` + `strcpy` 才是为自己“复印”一张永远有效的身份证。这个从**“引用”**到**“拷贝”**的思维转变，是我整个过程中**最重要、也最痛苦的一次思想飞跃**。



#### **第三步：蛮力实现——在赌博与天真中前进**



在解决了最核心的崩溃和所有权问题后，我就进入了“能跑就行”的蛮力阶段。这就是我那份“最终代码”的诞生过程。我做出了一系列的妥协：

1. **放弃扩容，选择信仰**：我面对动态数组的扩容需求，没有选择去学习并实现更复杂的`realloc`逻辑，而是简单粗暴地选择了`#define MAX_GROUP_SIZE 1000`。我用一个魔法数字，赌测试用例不会打败我。
2. **无视清理，选择遗忘**：我把所有的精力都放在了实现功能上，完全忽略了内存清理。我写的`groupAnagrams`函数变成了一个内存黑洞，吃进去的内存从不归还。我选择性地遗忘了`free`的存在。
3. **模糊处理，选择浪费**：在返回结果时，我虽然学会了正确的二级指针语法，但依然懒得去精确计算组数，而是直接按输入规模`strsSize`分配了最大可能的内存。

所以，我“最终的代码”是怎么诞生的？它是通过**一次致命的崩溃、一次核心思想的转变、以及一系列对复杂度的妥协和对内存管理的无视**而拼凑出来的。它能通过，不是因为它有多好，而是因为它在最关键的地方（所有权）做对了，而在其他地方的缺陷又恰好没有被测试用例的重拳戳穿。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定义常量以避免魔法数字
#define MAX_STRING_LENGTH 1000
#define MAX_GROUP_SIZE 1000

/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */
// djb2 哈希算法
char *sort(char *a){
    int len=strlen(a);
    for(int i=len-1;i>0;i--){
        for(int j=0;j<i;j++){
            if(a[j]-a[j+1]>0){
                char temp;
                temp=a[j];
                a[j]=a[j+1];
                a[j+1]=temp;
            }
        }
    }
    return a;
}
unsigned long hash_string(char *str) {
    unsigned long hash = 5381;
    int c;
    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c; // hash * 33 + c
    }
    return hash;
}
char*** groupAnagrams(char** strs, int strsSize, int* returnSize, int** returnColumnSizes) {
    typedef struct strlist{
        int count;
        char **val;
    }strlist;
    typedef struct Node{
        char *key;
        strlist class;
        struct Node *Next;
    }Node;
    Node **HashTable=(Node **)malloc(sizeof(Node *) * strsSize);
    for(int i=0;i<strsSize;i++){
        HashTable[i]=NULL;
    }
    //for循环，每一轮循环先复制，然后排序，求出哈希值，寻找对应的位置，插入或新建
    //遍历哈希表，不为NULL的就将每一个Node->class的val输出
    for(int i=0;i<strsSize;i++){
        char *cpystr=(char *)malloc(sizeof(char)*MAX_STRING_LENGTH);
        strcpy(cpystr,strs[i]);
        sort(cpystr);
        int index = hash_string(cpystr) % strsSize;
        Node *current=HashTable[index];
        while(current!=NULL){
            if(strcmp(cpystr,current->key)==0){
                current->class.val[current->class.count]=(char *)malloc(sizeof(char)*(strlen(strs[i])+1));
                strcpy(current->class.val[current->class.count], strs[i]);
                current->class.count++;
                break;
            }else{
                current=current->Next;
            }
        }
        if(current==NULL){
            Node *NewNode=(Node *)malloc(sizeof(Node));
            NewNode->key=(char *)malloc(sizeof(char)*MAX_STRING_LENGTH);
            strcpy(NewNode->key,cpystr);
            NewNode->class.count=1;
            NewNode->class.val=(char **)malloc(sizeof(char *)*MAX_GROUP_SIZE);
            NewNode->class.val[0]=(char *)malloc(sizeof(char)*(strlen(strs[i])+1));
            strcpy(NewNode->class.val[0], strs[i]);
            NewNode->Next=HashTable[index];
            HashTable[index]=NewNode;
        }


    }
    char ***result=(char ***)malloc(sizeof(char **)*strsSize);
    *returnColumnSizes=(int *)malloc(sizeof(int)*strsSize);
    *returnSize=0;
    for(int i=0;i<strsSize;i++){
        if(HashTable[i]!=NULL){        
            Node *current=HashTable[i];
            while(current!=NULL){
                result[*returnSize]=current->class.val;
                (*returnColumnSizes)[*returnSize]=current->class.count;
                (*returnSize)++;
                current=current->Next;
            }

        }
    }
    return result;
}
```



------



