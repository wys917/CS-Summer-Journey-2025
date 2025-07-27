# leetcode 230.二叉搜索树中第K小的元素

## 第一种方法，新开数组：

中序遍历这个树，将得到的结果储存到一个新的数组里，然后取他的第k个数，这种方法过于蠢笨，所以此处不演示。

## 第二种方法，递归实现：

不额外创建一个数组，而是在设计辅助函数的时候多传入一个int类型的整型指针，每次遍历令该整数自增，一旦其大小等于k，就终止递归，返回对应的值。刚刚试了一下，感觉不太行，主要是我这样搞得话根本就不能递归起来，inorder返回的值会被污染。

> 你终于意识到了这个递归函数的**返回值被污染**了。你既想让它返回-1来表示“还没找到”，又想让它在找到的时候返回`root->val`。这导致一旦递归深入，找到的正确值在返回上层的过程中，会被其他分支返回的-1给覆盖掉。

> 你这个失败的尝试，暴露了你对递归函数设计的一个核心知识点理解不清：**当你的递归函数需要在深层调用中“发现”一个最终答案并终止整个递归时，你不能依赖函数的返回值来传递这个答案。**

下面是**错误**的代码：

```C
int inorder(struct TreeNode* root,int k,int *idx){
    if(root==NULL){
        return -1 ;
    }
    inorder(root->left,k,idx);
    if(*idx==k){
        return root->val;
    }
    (*idx)++;
    inorder(root->right,k,idx);
    return -1;
}
int kthSmallest(struct TreeNode* root, int k) {
    int idx=1;
    int result=-1;
    result = inorder(root,k,&idx);
    return result;
}
```



正确的递归优化应该是怎样的？不是修改返回值，而是把`idx`和`result`都作为“状态”来维护。看好了，这是更优雅的递归写法：

```C
// 定义两个全局变量来充当“公告板”，记录状态
int g_rank = 0;    // 当前遍历到的排名
int g_result = 0;  // 最终的结果

/**
 * 辅助函数：中序遍历
 * 它的目标不是“返回”结果，而是“修改”全局变量
 * @param root 当前节点
 * @param k 目标排名
 */
void inorderTraversal(struct TreeNode* root, int k) {
    // 如果节点为空，或者我们已经找到了答案（排名超了），就没必要继续了
    if (root == NULL || g_rank >= k) {
        return;
    }

    // 1. 递归左子树
    inorderTraversal(root->left, k);

    // 2. 访问当前节点（“根”）
    // 排名自增
    g_rank++;
    // 检查是否找到了第k个
    if (g_rank == k) {
        // 找到了！在“公告板”上写下结果
        g_result = root->val;
        // 直接返回，没必要再遍历右子树了，这是个小优化
        return;
    }

    // 3. 递归右子树
    inorderTraversal(root->right, k);
}

/**
 * 主函数：总指挥
 */
int kthSmallest(struct TreeNode* root, int k) {
    // 每次调用主函数时，必须重置全局变量！
    // 这是使用全局变量时最重要、最容易犯错的地方！
    // LeetCode的测试用例会多次调用这个函数，不重置你第二次就死定了。
    g_rank = 0;
    
    // 发出遍历指令
    inorderTraversal(root, k);
    
    // 从“公告板”上读取最终结果并返回
    return g_result;
}
```

## 第三种方法，用迭代法:

只要当前节点非空，就入栈，如果当前节点是NULL并且栈非空，就弹栈，访问栈顶元素的右节点，然后重复。每次弹栈视为该元素被遍历，此时可以让计数变量cnt（初始值为0）自增，若自增后==k，直接返回该节点的值。

```C
int kthSmallest(struct TreeNode* root, int k) {
    int cnt=0;
    struct TreeNode **stack=(struct TreeNode **)malloc(sizeof(struct TreeNode * )*10001);
    int top=-1;
    struct TreeNode *cur=root;
    while(top!=-1||cur!=NULL){
        while(cur!=NULL){
            stack[++top]=cur;
            cur=cur->left;
        }
        cur=stack[top--];
        cnt++;
        if(cnt==k){
            free(stack);
            return cur->val;
        }
        cur=cur->right;
    }
    free(stack);
    return -1;
}
```

## 总结

今天表现还是可以的，至少做出来了，分析问题也开始变得有条理了,继续加油啊

2025年7月27日
