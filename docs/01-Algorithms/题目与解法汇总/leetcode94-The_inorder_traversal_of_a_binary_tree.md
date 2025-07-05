```C title="Inorder_Traversal_of_a_binary_tree.c" linenums="1" hl_lines="32-34"
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */


int* inorderTraversal(struct TreeNode* root, int* returnSize) {
    struct TreeNode** stack=(struct TreeNode**)malloc(sizeof(struct TreeNode*)*2000);
    *returnSize=0;
    int *res=(int *)malloc(sizeof(int )*2999);
    int top=-1;
    if(root==NULL){
        return NULL;
    }
    stack[++top]=root;
    root=root->left;
    while(top>-1){
        while(root!=NULL){
            stack[++top]=root;
            root=root->left;
        }
        root=stack[top--];
        res[(*returnSize)++]=root->val;
        root=root->right;
        if(root!=NULL){
            stack[++top]=root;
            root=root->left;
        }
    }
    return res;
}
```