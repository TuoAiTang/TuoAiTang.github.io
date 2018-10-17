---
layout:       post
title:        "解题日记Stack(一)"
subtitle:     "总得做两道"
date:         2018-10-17 21：17
author:       "Tuoc"
header-img:   "img/about-bg.jpg"
header-mask:  0.3
catalog:      true
tags:
    - C
    - 栈
    - LeetCode
---


# [496.Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/description/)
>You are given two arrays (without duplicates) nums1 and nums2 where nums1’s elements are subset of nums2. Find all the next greater numbers for nums1's elements in the corresponding places of nums2.
The Next Greater Number of a number x in nums1 is the first greater number to its right in nums2. If it does not exist, output -1 for this number.

当nums1和nums2相等时，实际上我们要为每一个nums2元素找到它右边比它大的第一个元素，很自然的，我们有一个O(n^2)的算法。我们用嵌套的两层循环，第一层推进nums2的元素，第二层先推进到该元素的位置，再向后找比他大的第一个元素。判断一个算法是否有简化空间的思路是判断它，是否引入了多余的运算。这里很显然，我们第一层已经推进到该元素，而在第二层居然要再次从头找到它的位置向后推进。我们为什么不能就近比较呢？

很自然的想到我们可以设置一个pre指针，和一个cur指针，判断cur的值如果大于pre的值那么就得到了一个结果，我们再让pre = cur,cur = cur->next,依此推进。cur的值如果小于pre的值那么，cur继续推进，我们需要再设置一个pre1 = cur。

对于12345的输入，我们这个思路一次遍历就可以很轻松的解决问题，我们的pre指针始终只有一个。
但当输入变成54321呢？我们可能要存储之前的所有pre元素，我们需要pre1, pre2, pre3...

经过上述思考，很容易的想到是栈的特性。

>【 栈的特性 】正常循环的情况下，数组的滚动（游标移动）是向后的，引入栈的时候，则可以有了向前滚动的机会（有了一定的反悔的机会），然后这样子就能够解决一些局部的问题（比如说，寻找相邻的大的数字）。由于栈还可以对于没有价值（已经发现了大的数字）的东西删除，这样子的遗忘功能，简化了搜索空间，问题空间。

代码：

```c
int* nextGreaterElement(int* findNums, int findNumsSize, int* nums, int numsSize, int* returnSize) {
    *returnSize = findNumsSize;
    int stack[1000];
    int ret[1000] = {0};
    int top = -1;
    int max = 0;
    int min = 0;
    for(int i = 0; i < numsSize; i ++){
        max = max > nums[i] ? max : nums[i];
        min = min < nums[i] ? min : nums[i];
    }
    int *map = (int*)malloc((max - min + 1) * sizeof(int));
    for (int i = 0; i < numsSize; ++i)
    {
        while(top != -1 && stack[top] < nums[i])
            map[stack[top--]] = nums[i];

        stack[++top] = nums[i];
    }

    while(top > -1)
        map[stack[top--]] = -1;

    for (int i = 0; i < *returnSize; ++i)
        ret[i] = map[findNums[i]];

    free(map);

    return ret;
}
```

提交后我们看到一个错误，`load of null pointer of type 'const int'`,去看一下Dev C++ 的warning。

>[Warning] function returns address of local variable [-Wreturn-local-addr]吧 

发现我们返回的是一个本地变量，ret，函数结束后，ret被回收，ret这个指针也无效了。

我们做出修改：
```c
int *ret = (int*)malloc(findNumsSize * sizeof(int));
```
通过动态内存分配的指针，函数结束后不回收。所以这样做是合法有效的。


# [844. Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/description/)


第一次提交：
```c
char* afterBackspace(char* S){
    char stack[200];
    int top = -1;
    while(*S != '\0'){
        if(*S == '#'){
            if(top != -1)
                top--; 
        }
        else
            stack[++top] = *S;
        S ++;
    }   
    int size = top + 1; 
    // char* ret = (char*)calloc(size + 1, sizeof(char));   
    printf("size:%d\n", size);
    char* ret = (char*)calloc(size, sizeof(char));  
    for (int i = 0; i < size; ++i)
        ret[i] = stack[i];      
    // ret[size] = '\0';
    printf("size of ret:%d\n", strlen(ret));    
    return ret;
}

int backspaceCompare(char* S, char* T) {
    char *s = afterBackspace(S);
    char *t = afterBackspace(T);
    if(strcmp(s, t) == 0)
        return 1;       
    return 0;
}
```
思路很简单，但结果最后一个case过不了。
`103 / 104 test cases passed.`
这里涉及到一个malloc/calloc的陷阱！


# malloc/calloc陷阱

malloc并不一定分配给你确切的内存，它有可能比你的需要稍大一些。  

测试用例：
```c
void randCharFill(char* s, int s_size){
    for (int i = 0; i < s_size; ++i)
        s[i] = 65 + rand() % 40;
}

int size = 200;
char* p = (char*)malloc((size) * sizeof(char));
randCharFill(p, size);
printf("%s\n", p);
printf("len of p:%d\n", strlen(p)); 
printf("p[size]:%c\n", p[size]);
```
输出结果：  

>B\OUJEggCYZZB\BLdW\e\`EWbMWVegPHGLS\^a\TdOXLCNbYV\`N]\EWffTDVJSed_CI[AWYIGFKJK_GVb]\^XEceAGQL]Yh[DRSgCJVbdhSYKR[b[VZEa_\^RNRa[KBedHPO\`M__VEG_\`HRRHbXZ^JgV]W[[_NIU\`CPKhYR]DdBC_LeOUeV]h]EBONhSSU]H\INIDHV_RNcb8恟漮<br>
len of p:206<br>
p[size]:b<br>

`printf("%s", s)`这个语境下，是根据终止符来确定打印字符串终止的，strlen函数也是一样。很明显malloc实际分配了206个字节大小的空间，而这最后6个字节大小是没有被我所设置的randCharFill函数初始化的，那么可以看到第201个字节位置存放的是b,当我们下一次执行程序时我们也许看到的可能是`?`,`c`,`d`,也可能是另外一个我们无法预知的结果，最后4个字节大小的空间里面甚至存储了两个汉字。

由此做出改进：

```c
char* p = (char*)malloc((size + 1) * sizeof(char));
p[size] = '\0';
```

我们始终确保malloc函数分配稍大的内存，并在我们期待的终止位置填充终止符`\0`,那么我们可以确保我们通过动态内存申请创建的字符串，总是符合我们的要求。


本题的最后一个case，栈内剩余元素是40，也就是size=40, 经测试，我们发现此时的确分配的实际内存是46！
这个test case真的鬼精鬼精的！所以那6个字节的内容，无法预知的那6个内容，当然不可能相等。

按照注释修改即可通过！