# LeetCode 98: 验证二叉搜索树

!!! info "题目"
        给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

        **有效** 二叉搜索树定义如下：

        - 节点的左子树只包含 **小于** 当前节点的数。
    
        - 节点的右子树只包含 **大于** 当前节点的数。
   
        - 所有左子树和右子树自身必须也是二叉搜索树。

        **示例 1：**
        ![alt text](https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg)

        > 输入：root = [2,1,3]

        > 输出：true

        **示例 2：**
        ![alt text](https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg)

         > 输入：root = [5,1,4,null,null,3,6]

        > 输出：false

        > 解释：根节点的值是 5 ，但是右子节点的值是 4 。

一开始拿到这道题的时候，直接动手使用递归完成了，写下了以下错误代码：

```  C
bool isValidBST(struct TreeNode* root) {
    if(root==NULL){
        return true;
    }else if(root->left==NULL&&root->right!=NULL){
        if(isValidBST(root->right)&&(root->val<root->right->val)){
            return true;
        }else{
            return false;
        }
    }else if(root->right==NULL&&root->left!=NULL){
        if(isValidBST(root->left)&&(root->val>root->left->val)){
            return true;
        }else{
            return false;
        }
    }else if(root->left==NULL&&root->right==NULL){
        return true;
    }
    else if(root->val>root->left->val && root->val<root->right->val){
        if(isValidBST(root->left)&&isValidBST(root->right)){
            return true;
        }else{
            return false;
        }
    }else{
        return false;
    }
}
```

这段代码错误的原因很明显，只检查了当前节点与左右节点的大小关系，没有考虑其与更高节点的大小关系，因此没有通过部分测试点

---



经过思考后，想出了以下正确的递归，使用了一个辅助的递归函数，并传入了两个long类型的参数，即上下界。值得注意的是，第一次传入的root没有上下界的要求，所以需要利用`#include <limits.h>`中的`LONG_MIN`和`LONG_MAX`来获取最小值和最大值。并且这两个东西的数据类型是`long`，所以要注意是`long lower`而不是`int lower`

``` C
//方案一——递归法
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
#include <limits.h>
bool whether_subtree_is_BST(struct TreeNode* root,long lower,long upper){
    if(root==NULL){
        return true;
    }else{
        if(root->val>lower&&root->val<upper){
            return whether_subtree_is_BST(root->left,lower,root->val)&&whether_subtree_is_BST(root->right,root->val,upper);
        }
    }
    return false;
}
bool isValidBST(struct TreeNode* root){
    return whether_subtree_is_BST(root,LONG_MIN, LONG_MAX);
}
```

写完上述代码后，leetcode已经通过了，于是我又研究了一下答案，希望能找到更好的实现方式。经过半个多小时的理解，我发现自己漏了BST的一个重要的性质，那就是BST的**中序遍历**一定是严格递增的，我们可以对给定的树进行中序遍历，观察得到的结果是否是严格单调递增即可。

---



然后我就开始写了，结果写出了这样一大坨错误百出的代码，哎，果然还是要天天码才会有手感，四五天没写代码就变成这样了，伤心

```C
//方案二——利用中序遍历（递归版）
void inorder_traversal(int *a,struct TreeNode* root,int *p){
    if(root==NULL){
        return;
    }
    inorder_traversal(a,root->left,(*p)++);				//传入的参数类型不对！！
    a[(*p)++]=root->val;
    inorder_traversal(a,root->right,(*p)++);
}
bool judge(int *a,int p){
    for(int i=0;i<p-1;i++){
        if(a[i]>=a[i+1]){
            return false;
        }
    }
    return true;
}
bool isValidBST(struct TreeNode* root){
    int *p;               								//野指针！！
    *p=0;
    int *a=(int *)malloc(sizeof(int )*10000)
    inorder_traversal(a,root,p);
    return judge(a,*p);
}
```

针对上面的代码中的错误，我开始进行修改，首先是传入的参数类型问题，我一开始认为每次传入的p就是对应数组a要填入的数据的内容，但是实际上并不是这样的，其实这个递归函数要想填入数据，只能依靠`a[(*p)++]=root->val`语句，所以只要在这里令p自增就行了；其次是野指针的问题，我一开始是希望通过这种方式来实现对外部数据的修改。其实可以换一个思路，使用&来取地址就行了。下面是修改后的代码：

```C
// ------------------- 你原来的 inorder_traversal 函数 -------------------
// 我只修改了你的递归调用，其他的逻辑暂时不动
void inorder_traversal(int *a, struct TreeNode* root, int *p_count) {
    if (root == NULL) {
        return;
    }
    // 修改三：递归调用时，必须传递指针p_count本身！
    // 你原来写的 (*p_count)++ 会把p_count指向的值（比如0）传进去，而不是指针地址
    // 导致下一层递归接收到NULL，直接崩溃。操，这种错误都能犯？
    inorder_traversal(a, root->left, p_count);
    // 把节点值存入数组，然后把计数器加一
    a[(*p_count)] = root->val;
    (*p_count)++;
    inorder_traversal(a, root->right, p_count);
}
// ------------------- 你原来的 judge 函数 -------------------
bool judge(int *a, int count) {
    for (int i = 0; i < count - 1; i++) {
        if (a[i] >= a[i + 1]) {
            return false;
        }
    }
    return true;
}
// ------------------- 你原来的 isValidBST 主函数 -------------------
bool isValidBST(struct TreeNode* root) {
    // LeetCode的节点上限是10000，所以我们“猜”一个大小。
    // 这在真实工程里是极其傻逼的行为，但为了满足你这个烂方法，只能这么干。
    // 修改二：正确的内存分配。用sizeof(int)，并且给一个具体的、猜出来的数量。
    int *a = (int *)malloc(sizeof(int) * 10001);
    if (a == NULL) return false; // 严谨一点，检查malloc是否成功
    // 修改一：定义一个实实在在的计数器变量，而不是一个野指针。
    // 你原来 int *p; *p=0; 的写法，是100%的段错误，教科书级的反面案例。
    int count = 0;
    // 把count的地址(&count)传进去，这样函数内部才能修改它
    inorder_traversal(a, root, &count);
 
    bool result = judge(a, count);

    // 别忘了释放你申请的内存，不然就是内存泄漏。
    // 你看，用数组多几把麻烦。
    free(a);
    
    return result;
}
```

---



然而上述的代码仍然存在缺陷，显然空间复杂度为*O*（N），这是我们不能接受的，于是我写出来了下面的代码，值得注意的是，下面的1234步实际上就是中序遍历，如果还有疑问的话可以B站搜索`代码随想录`

```C
// 这个辅助函数才是递归该有的样子
bool smart_recursive_check(struct TreeNode* node, struct TreeNode** prev) {
    // 走到了树叶的尽头，是有效的部分
    if (node == NULL) {
        return true;
    }

    // 1. 先一路向左，递归检查左子树
    if (!smart_recursive_check(node->left, prev)) {
        return false;
    }

    // 2. 处理当前节点（中序遍历的核心访问点）

    if (*prev != NULL && (*prev)->val >= node->val) {
        return false;
    }

    *prev = node;

    // 4. 最后递归检查右子树
    return smart_recursive_check(node->right, prev);
}

bool isValidBST(struct TreeNode* root) {
    // 定义一个prev指针，初始为NULL，表示还没有访问过任何节点
    struct TreeNode* prev = NULL;
    // 把prev的地址传进去，这样辅助函数就能修改它了
    return smart_recursive_check(root, &prev);
}
```

---



这是迭代版，要记住！！！

```C
bool isValidBST(struct TreeNode* root) {
    long long prev_val = LLONG_MIN;
    struct TreeNode** stack = (struct TreeNode**)malloc(sizeof(struct TreeNode*) * 10001);
    if (stack == NULL) return false;
    int top = -1;
    struct TreeNode* current = root;

    // 看！这！里！
    // 简洁！明了！正确！
    while (current != NULL || top != -1) { //特别是这里的current!=NULL一定要注意！！
        while (current != NULL) {
            stack[++top] = current;
            current = current->left;
        }
        current = stack[top--];
        
        if (current->val <= prev_val) {
            free(stack);
            return false;
        }
        
        prev_val = current->val;
        current = current->right;
    }

    free(stack);
    return true;
}
```

这道题花了我很长时间，说明我对一些基础的操作还是存在很大的问题的，要继续加油啊！！