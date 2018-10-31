---
layout:       post
title:        "解题日记Stack(二)"
subtitle:     "TwoStacks/顺序/结果"
date:         2018-10-23 21：32
author:       "Tuoc"
header-img:   "img/about-bg.jpg"
header-mask:  0.3
catalog:      true
tags:
    - C
    - 栈
    - LeetCode
---

## 栈！


为什么要用栈来解决问题？
什么时候，哪些场景需要用到栈？
用栈怎么简化算法，降低时间复杂度？
用栈降低时间复杂度，解决问题的关键是什么？

我在解题的过程中，总会很自然的想到这些问题。

要理解以上问题，我们不妨再回顾一下栈的特性。

>正常循环的情况下，数组的滚动（游标移动）是向后的，引入栈的时候，则可以有了向前滚动的机会（有了一定的反悔的机会），然后这样子就能够解决一些局部的问题（比如说，寻找相邻的大的数字）。由于栈还可以对于没有价值（已经发现了大的数字）的东西删除，这样子的遗忘功能，简化了搜索空间，问题空间。

毫无疑问，当一个算法完全不进行**多余**的运算，那么它是一个时间复杂度最低的算法。但我们往往会对一些结果进行**重复**的计算，那么栈的引入就是为了解决这样的问题，栈**存储**了一些**重要**的运算结果，用于和接下来的元素进行**比较**。

具体来说，我认为解题的关键在于以下几个点:


- **入栈应该维持一个怎样的顺序(Ascending?Descending?)***（重中之重）*
- 出栈时调整结果的策略
- 遍历的方向（从左到右？从右到左？）

leetcode关于`stack`的题蛮多的，拿下面这些题来作例子吧。



## [456. 132 Pattern](https://leetcode.com/problems/132-pattern/description/)
```c
bool find132pattern(int* nums, int numsSize) {

    if(numsSize < 3)
        return 0;

    int* stack = (int*)malloc(numsSize * sizeof(int));
    int top = -1;

    int* min = (int*)malloc(numsSize * sizeof(int));
    min[0] = nums[0];

    for (int i = 1; i < numsSize; ++i)
        min[i] = min[i-1] < nums[i] ? min[i -1] : nums[i];

    for (int j = numsSize - 1; j >= 0; j--)
    {
        if(nums[j] > min[j]){
            while(top != -1 && stack[top] <= min[j])
                top--;
            if(top != -1 && nums[j] > stack[top])
                return true;
            stack[++top] = nums[j];
        }
    }

    free(stack);
    free(min);

    return 0;
}
```
先来一趟遍历，对于每个`i`,找到**到i为止**最小的元素，并存储为`min[i]`。


**从右向左遍历**，对每个有**潜在可能**成为`132`模式的`j`（满足`num[j] > min[j]`），不断弹出，判断是否存在比`num[j]`更小的元素，如果有，那么找到了`132`.如果遇到一个栈顶元素大于 `num[j]`就应该停止，因为栈内的其他元素都将比`num[j]`大，此时入栈，维护了这个**递增**的序列。可以这么说，这个栈**保留**了这个**潜在可能**的`j`**右侧**的**所有**元素，并且由**栈顶到栈底**是一个**递增**的序列。


## [735. Asteroid Collision](https://leetcode.com/problems/asteroid-collision/description/)

```c
int* asteroidCollision(int* asteroids, int asteroidsSize, int* returnSize) {

    int* stack = (int*)malloc(asteroidsSize * sizeof(int));
    int top = -1;
    for (int i = 0; i < asteroidsSize; ++i)
    {
		if(top != -1 && asteroids[i] < 0 && stack[top] > 0){
			
			if(abs(asteroids[i]) > abs(stack[top])){
				while(abs(asteroids[i]) > abs(stack[top]) && top != -1){
					if(stack[top] < 0)
						break;
					top--;
				}
				if(top == -1 || stack[top] < 0){
					stack[++top] = asteroids[i];
					continue;
				}
			}

			if(abs(asteroids[i]) == abs(stack[top]))
				top --;	
		}
		else
			stack[++top] = asteroids[i];
    }
    *returnSize = top + 1;
    int* ret = (int*)malloc((*returnSize) * sizeof(int));
    for (int i = 0; i < *returnSize; ++i)
		ret[i] = stack[i];
	free(stack);
	return ret;
}
```
在添加一个元素前，之前的序列已经稳定。是一个`stable`的序列。
我们直接用一个`stack`来存储已经稳定的序列，让它从**空栈**开始添加元素。
只有当前元素为负值，上一个元素为正值时它会发生爆炸。
在这种情况下：如果当前元素绝对值大于前一个元素 ，也就是栈顶元素，那么栈顶元素弹出，需要注意的是，如果推进时栈顶元素小于零，那么停止弹出，序列已经稳定，这时将当前元素push进栈。

## [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/description/)
```c

int trap(int* height, int heightSize) {
    int* stack = (int*)malloc(heightSize * sizeof(int));
    int top = -1;
    int ans = 0;
	for (int i = 0; i < heightSize; ++i){
		while(top != -1 && height[i] > height[stack[top]]){
			int bottom = stack[top--];
			//the very biginning can not trap rain
			if(top == -1)
				break;
			int bar = min(height[i], height[stack[top]]) - height[bottom];
			//if bar == 0, process forward, distance would increase
			int distance = i - stack[top] - 1;
			ans += distance * bar;
		}
		stack[++top] = i;
	}
	free(stack);
    return ans;
}
```

一趟遍历，我们维护一个由栈底到栈顶递增的栈，当遇到当前元素高于栈顶元素时我们计算蓄水量并弹出栈内元素，当弹出所有小于当前元素时，入栈当前元素，保持栈内的顺序。

## [394. Decode String](https://leetcode.com/problems/decode-string/description/)
```java
class Solution {
    public String decodeString(String s) {
        Stack<Integer> countStack = new Stack<>();
    	Stack<String> resStack = new Stack<>();
    	int idx = 0;
    	int count = 0;
    	String res = "";
    	while(idx < s.length()){
    		count = 0;
    		if(Character.isDigit(s.charAt(idx))){
    			while(Character.isDigit(s.charAt(idx)))
    				count = 10 * count + s.charAt(idx++) - '0';
    			countStack.push(count);
    		}else if(s.charAt(idx) == '['){
    			resStack.push(res);
    			res = "";
    			idx++;
    		}else if(s.charAt(idx) == ']'){
    			int repeatTimes = countStack.pop();
    			StringBuilder sb = new StringBuilder(resStack.pop());
    			for (int i = 0; i < repeatTimes; i++)
    				sb.append(res);
    			res = sb.toString();
    			idx++;
    		}else{
    			res += s.charAt(idx++);
    		}
    	}
        return res;
    }
}
```
两个栈分别处理重复次数和字符串
遇到`[`时，将上一个res入栈，让res重置；
遇到`]`时，将count弹出，并借助sb来给现有的res添加重复元；
遇到最后一个`]`后，res即为完整的解码字符串；

## [224. Basic Calculator](https://leetcode.com/problems/basic-calculator/description/)
```c
int isDigit(char c){
    int dis = c - '0';
    return (dis >= 0 &&  dis <= 9) ? 1 : 0;
}
int calculate(char* s) {
	int n = strlen(s);
	int result = 0;
	int number = 0;
	int sign = 1;
	int top = -1;
	// we don't need to push if no '(' contained
	//else we push n / 2 times without pop at most
	//so set the size to half of the length
	int* stack = (int*)malloc(n / 2 * sizeof(int));
    for (int i = 0; i < n; ++i)
    {
    	char c = s[i];
    	if(isDigit(c))
    		number = 10 * number + c - '0';
    	else{
    		switch(c){
	    		case '+':
	    			result += sign * number;
	    			sign = 1;
	    			number = 0;
	    			break;
	    		case '-':
	    			result += sign * number;
	    			sign = -1;
	    			number = 0;
	    			break;
	    		case '(':
	    			stack[++top] = result;
	    			stack[++top] = sign;
	    			//reset
	    			result = 0;
	    			sign = 1;
	    			break;
	    		case ')':
	    			result += sign * number;
	    			//firt-sign, second-result before
	    			result *= stack[top--];
	    			result += stack[top--];
	    			number = 0;
	    			break;

	    		default:
	    			break;
    		}
    	}
	}
	if(number != 0)
		result += sign * number;
	free(stack);
    return result;
}
```
与上一题类似的，我们的stack只存储**这一层**括号的**结果**和这一层**之前**的符号，遇到其他运算符先处理之前的number,并让符号变化。


## [636. Exclusive Time of Functions](https://leetcode.com/problems/exclusive-time-of-functions/description/)

```c
int* getParseLog(char const * s){
	int* ret = (int*)calloc(3, sizeof(int));

	while(*s != ':'){
		ret[0] = 10 * ret[0] + *s - '0';
		s ++;
	}
	s ++;
	if(*s == 's'){
		ret[1] = 1;
		s += 6;
	}else{
		ret[1] = 0;
		s += 4;
	}
	while(*s != '\0'){
		ret[2] = 10 * ret[2] + *s - '0';
		s ++;
	}
	return ret;
}

int* exclusiveTime(int n, char** logs, int logsSize, int* returnSize) {
	*returnSize = n;
    int* ret = (int*)calloc(n, sizeof(int));
    int* stack = (int*)malloc((logsSize / 2) * sizeof(int));
    int top = -1;
    int pre = 0;
    int* log;
    for (int i = 0; i < logsSize; ++i)
    {
    	log = getParseLog(logs[i]);
    	if(log[1] == 1){
    		if(top != -1)
    			ret[stack[top]] += log[2] - pre;
    		stack[++top] = log[0];
    		pre = log[2];
    	}else{
    		ret[stack[top]] += log[2] - pre + 1;
    		top--;
    		pre = log[2] + 1;
    	}
    	free(log);
    }
    free(stack);
    return ret;
}
```
我们先定义一个函数解析我们每一个log字符串，三个位置分别为`id, start/end flag,time`。


我们的栈存储每个执行函数的`id`，每次遇到一个函数的开始，我们让栈顶`id`对应的函数,也就是这个函数的主调函数的运行时间加上它开始时间与`pre`的差值，这个差值实际上就是外层函数的**独占**时间。接着入栈当前函数` id`,推进`pre`，让它与当前时间相同。再遍历下一个`log`。


当遇到一个函数结束时，此时栈顶元素是与这个函数匹配的，意味着这个函数的生命周期结束 ，增加它的独占运行时间，并出栈，保证我们的栈内只存储那些生命周期还未结束的函数。推进`pre`到当前时间`+1s`



## [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/description/)

```c
int largestRectangleArea(int* heights, int heightsSize) {
    int* stack = (int*)malloc(heightsSize * sizeof(int));
    int top = -1;
    int maxArea = 0;
    int count = 0;
    for (int i = 0; i < heightsSize; ++i)
    {
    	count = 0;
    	//ensure ascending sequence in the stack
    	while(top != -1 && heights[i] < stack[top]){
    		int t = stack[top--];
    		count++;
    		maxArea = max(t * count, maxArea);
    	}
    	//push the element replaced by heights[i] popped just before
    	while(count--)
    		stack[++top] = heights[i];
    	//push the current element
    	stack[++top] = heights[i];
    }
    //calculate the rest in the stack
    count = 0;
    while(top != -1){
    	int t = stack[top--];
    	count++;
    	maxArea = max(t * count, maxArea);
    }
    return maxArea;
}
```


1、如果已知height数组是升序的，应该怎么做？

比如1,2,5,7,8

那么就是(1\*5) vs. (2\*4) vs. (5\*3) vs. (7\*2) vs. (8\*1)

也就是max(height[i]\*(size-i))

2、使用栈的目的就是构造这样的升序序列，按照以上方法求解。

但是height本身不一定是升序的，应该怎样构建栈？

比如2,1,5,6,2,3

（1）2进栈。s={2}, result = 0

（2）1比2小，不满足升序条件，因此将2弹出，并记录当前结果为2\*1=2。

将2替换为1重新进栈。s={1,1}, result = 2

（3）5比1大，满足升序条件，进栈。s={1,1,5},result = 2

（4）6比5大，满足升序条件，进栈。s={1,1,5,6},result = 2

（5）2比6小，不满足升序条件，因此将6弹出，并记录当前结果为6\*1=6。s={1,1,5},result = 6

2比5小，不满足升序条件，因此将5弹出，并记录当前结果为5\*2=10（因为已经弹出的5,6是升序的）。s={1,1},result = 10

2比1大，将弹出的5,6替换为2重新进栈。s={1,1,2,2,2},result = 10

（6）3比2大，满足升序条件，进栈。s={1,1,2,2,2,3},result = 10

栈构建完成，满足升序条件，因此按照升序处理办法得到上述的max(height[i]\*(size-i))=max{3\*1, 2\*2, 2\*3, 2\*4, 1\*5, 1\*6}=8<10


更多stack例题解答可以参考[我的解答](https://github.com/TuoAiTang/LeetCode),也欢迎访问我下方的的Leetcode主页，晚安~





