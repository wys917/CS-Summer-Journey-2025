# LeetCode 3: 无重复字符串的最长子串
!!! info "题目"
    给定一个字符串 s ，请你找出其中不含有重复字符的 最长 子串 的长度。


    示例 1:

    输入: s = "abcabcbb"

    输出: 3 

    解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。

    示例 2:

    输入: s = "bbbbb"

    输出: 1

    解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。

    示例 3:

    输入: s = "pwwkew"

    输出: 3

    解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。

        请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
    

    提示：

    0 <= s.length <= 5 * 104

    s 由英文字母、数字、符号和空格组成
我一开始的想法很朴素，就是搞一个双重循环，寻找可能满足条件的子串，如果成立则进入第二层筛选，但是这种方法极差，时间复杂度达到惊人的O(N^3):
```c
bool find(char *s,int start,int end){
    for(int i=start;i<end;i++){
        for(int j=i+1;j<end;j++){
            if(s[i]==s[j]){
                return false;
            }
        }
    }
    return true;
}
int lengthOfLongestSubstring(char* s) {
    int i,j;
    int res=1,max=1;
    int len=strlen(s);
    if(len==0){
        return 0;
    }
    for(int i=0;i<len;i++){
        j=i+1;
        while(j<len&&s[j]!=s[i]){
            res++;
            j++;
        }//找到了可能的字符串，s[i]---s[j-1]
        while(!find(s,i,j)&&(j-i+1)>max){
                j--;
        }
        res=j-i;
        if(res>max && find(s,i,j)){
            max=res;
        }
        res=1;

    }
    return max;
}
```
??? note "为什么很差？"
    1.  find 函数：性能灾难 🐢
    ```
    bool find(char *s,int start,int end){
        for(int i=start;i<end;i++){
            for(int j=i+1;j<end;j++){ // 经典 O(N^2)
                if(s[i]==s[j]){
                    return false;
                }
            }
        
        return true;
    }
    ```

    就为了判断一个长度的子串里有没有重复字符，直接上了两层循环,一个长度为 k 的子串，这个函数要跑 O(k 
    2
    ) 的时间。LeetCode 的测试用例稍微长一点，这个函数就直接超时。

    正确姿势：判断重复字符用哈希表（或者一个大小为128的数组来模拟哈希表，因为字符最多就128个ASCII码）遍历一遍子串，把出现过的字符放进哈希表，每次放入前检查一下是否已存在。这样时间复杂度就是 O(k)，比我那个 O(k 
    2
    ) 不知道高到哪里去了。

    2. 主函数 lengthOfLongestSubstring：逻辑的迷宫 😵
   

    第一步：一个莫名其妙的 while 循环

    ```c
    j=i+1;
    while(j<len && s[j]!=s[i]){
        res++;
        j++;
    }
    // 找到了可能的字符串，s[i]---s[j-1]
    ```
    这个循环的逻辑是：从 s[i] 开始，只要后面的字符不等于 s[i]，就继续前进。

    这是什么逻辑？
    对于字符串 "abcabcbb"，当 i=0 (s[i] = 'a')，j 会一直走到 s[3] 也就是第二个 'a' 的位置才停下。找到的子串是 "abc"。
    但对于字符串 "pwwkew"，当 i=1 (s[i] = 'w')，j 在 s[2] 的位置就停下了，因为 s[2] 也是 'w'。找到的子串是 "w"。压根没考虑 "wkew" 这种情况。只检查了与第一个字符的重复，这有什么用？

    第二步：一个更愚蠢的 while 循环

    
    ```c
    while(!find(s,i,j)&&(j-i+1)>max){
        j--;
    }
    ```
    因为第一步找的子串是错的，所以现在想往回找补。用那个性能奇差的 find 函数，从右边一个一个地缩减字符，直到子串里没有重复的为止。

    这操作的复杂度是多少？外层有个 for 循环 O(N)，里面这个 while 循环最坏情况下也要跑 O(N) 次，每次循环还调用一次你那个 O(N 
    2
    ) 的 find 函数。整体复杂度奔着 O(N 
    4
    ) 就去了，就算平均下来也是妥妥的 O(N 
    3
    )。

    第三步：一次多余的 if 判断

    
    ```c
    res=j-i;
    if(res>max && find(s,i,j)){
        max=res;
    }
    ```
    前面那个 while 循环结束的条件不就是 find(s,i,j) 返回 true 吗（或者长度不够长）？

查阅了相关资料后，发现这道题用滑动窗口解答是最合适的：

正确的解法：滑动窗口 (Sliding Window)

核心思想：

用两个指针，left 和 right，构成一个“窗口”。
right 指针不断向右移动，扩大窗口，每次都把新字符装进窗口。
用一个哈希表（或数组）记录窗口内每个字符出现的最新位置。

当 right 指向的字符在窗口内已经存在时，说明出现了重复。此时，不能再扩大窗口了，需要移动 left 指针，把窗口收缩。left 应该一步到位，直接移动到重复字符上一次出现位置的下一个位置。

在每次移动 right 之后，都计算一下当前窗口的长度 (right - left + 1)，并更新最大长度。

代码实现

```c
#include <string.h>
#define max(a, b) ((a) > (b) ? (a) : (b))

int lengthOfLongestSubstring(char* s) {
    // map 用来存储字符上一次出现的位置+1，+1是为了区分未出现(0)和出现在第0位的情况
    int map[128] = {0}; 
    int max_len = 0;
    int left = 0; // 窗口左边界

    int n = strlen(s);
    for (int right = 0; right < n; right++) {
        char current_char = s[right];

        // 如果当前字符在窗口内已经出现过
        // map[current_char] > left 表示该字符的上一次出现位置在当前窗口的左边界或其右侧
        if (map[current_char] > left) {
            left = map[current_char]; // 直接把左边界移动到重复字符的下一个位置
        }

        // 更新当前字符的位置（存入 right + 1，作为下一次的 left）
        map[current_char] = right + 1; 

        // 更新最大长度
        max_len = max(max_len, right - left + 1);
    }

    return max_len;
}
```

时间复杂度：O(N)。因为 left 和 right 两个指针都只从头到尾遍历了一遍字符串。

空间复杂度：O(K)，其中 K 是字符集的大小（ASCII码就是128），可以看作是 O(1) 的常数空间。

