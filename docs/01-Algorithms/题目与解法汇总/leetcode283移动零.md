# leetcode283移动零
!!! info "题目"
    给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
    请注意 ，必须在不复制数组的情况下原地对数组进行操作。

    示例 1:

    输入: nums = [0,1,0,3,12]

    输出: [1,3,12,0,0]
这道题我一开始的想法是使用稳定的排序算法（略加改装），比如冒泡排序，插入排序。但是有点太复杂了，时间复杂度为O(N^2)，如下：
```c
#include <stdio.h>

void moveZeroes_InsertionSort(int* nums, int numsSize) {
    if (numsSize <= 1) {
        return;
    }

    // i 从 1 开始，代表“未排序”部分的第一个元素
    for (int i = 1; i < numsSize; i++) {
        
        int key = nums[i]; // 这是我们这次要处理的牌

        // 如果这张牌是0，我们根本不关心它，直接跳过，让它待在后面
        if (key == 0) {
            continue;
        }

        // j 是“已排序”部分的末尾
        int j = i - 1;

        // 现在 key 是一个非零数，我们要给它找位置
        // 我们需要把前面所有是0的牌都往后挪
        // 只要 j 还没越界，并且指着一张0牌，就把它往后挪
        while (j >= 0 && nums[j] == 0) {
            nums[j + 1] = nums[j]; // 0往后挪
            j--;
        }

        // 循环结束，j+1 就是这个非零数 key 该插入的位置
        nums[j + 1] = key;
    }
}

// (这里可以放一个main函数来测试)
int main() {
    int arr[] = {0, 1, 0, 3, 12};
    int size = sizeof(arr) / sizeof(arr[0]);
    
    moveZeroes_InsertionSort(arr, size);
    
    printf("Result: ");
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n"); // 应该输出: 1 3 12 0 0 
    
    return 0;
}
```


如上可见，这是非常垃圾的答案，所以开始寻找别的解法，最终找到了双指针法，第一版代码如下，用一个while循环让指针i跳过所有的0，找到第一个非零数。用另一个while循环让指针j跳过所有的非零数，找到第一个零。然后把它们交换：
```c
void swap(int *nums,int i,int j){
    int temp=nums[i];
    nums[i]=nums[j];
    nums[j]=temp;
}
void moveZeroes(int* nums, int numsSize) {
    int i=0,j=0;
    while(j<numsSize&&i<numsSize){
        while(i<numsSize&&nums[i]==0){
            i++;
            if(i>=numsSize){
                return;
            }
        }
        while(j<numsSize&&nums[j]!=0){
            j++;
            if(j>=numsSize){
                return;
            }
        }

        swap(nums,i,j);
        i++;
        j++;

    }
}
```
这个思路，本质上是在找一个“错位的非零数”和一个“错位的零数”，然后把它们换过来。这个过程为了维持顺序，要求j必须在i的前面。代码里虽然没有显式判断j < i，但因为j找0、i找非0，所以第一次交换时j总会在i前面。
虽然逻辑上能走通，但它绕了天大的一个弯子。并且效率低下，存在重复扫描，这套指针移动的机制，i和j在内层循环里前进，在外层循环的结尾又各自前进一次，这会导致指针在数组的某些部分来回移动，进行不必要的重复扫描。

正确的思路：双指针法（铲雪车模型）

需要两个指针：

慢指针 slow (或者叫 insertPos)：

它的作用，是指向下一个非零元素应该被放置的位置。一开始索引为0。

快指针 fast (或者叫 i)：

它的作用，是去遍历整个数组，寻找非零元素。它从头走到尾，不放过任何一个元素。

算法流程：

初始化 slow = 0。

fast 指针从 0 遍历到数组末尾。

在遍历过程中，只要 fast 指针发现了一个非零元素 (nums[fast] != 0)：

就把这个非零元素“铲”到 slow 指针指向的位置上，即 nums[slow] = nums[fast]。

然后，让 slow 指针向前移动一格，slow++，为下一个非零元素腾出位置。

当 fast 指针走完全程后，所有非零元素就已经被我们紧凑地、且按原有顺序地移动到了数组的开头。slow 指针此时就停在了最后一个非零元素的下一个位置。

最后，从 slow 的位置开始，把数组剩下的所有位置全填成0。

```c
void moveZeroes(int* nums, int numsSize) {
    int slow = 0; // 指向下一个非零元素该放的位置

    // 第一次遍历：把所有非零元素往前移动
    for (int fast = 0; fast < numsSize; fast++) {
        if (nums[fast] != 0) {
            nums[slow] = nums[fast];
            slow++;
        }
    }

    // 第二次遍历：把 slow 之后的所有位置都补上零
    for (int i = slow; i < numsSize; i++) {
        nums[i] = 0;
    }
}
```
这道题的收获就是不要思维定式了，要以结果为导向，并不是一定要交换元素来实现结果的，像这样两次遍历会更方便也更好理解
