---
layout:       post
title:        "链表题解之TwoPointers（LeetCode)"
subtitle:     "TwoPointers的应用举例"
date:         2018-10-15
author:       "Tuoc"
header-img:   "img/in-post/c-simple.jpg"
header-mask:  0.3
catalog:      true
tags:
    - C
    - 指针
    - 链表
    - LeetCode
---

## TwoPointers  
  
### 概念
TwoPointers 指的是**一快一慢**的两个指针去推进一个链表，我把它分为两类。
- 第一类：起点不一样，fast比slow先走`n`步
- 第二类：步长不一样，fast比slow推进的快，例如`fast = fast->next->next; slow = slow->next`,fast的**步长**为`2`，而slow的**步长**为`1`
- 当然，还有一个另类我也把它归为TwoPointers，后面我将会举一个有趣的例子

### TwoPointers可以解决哪些问题？
1. 找到链表的**中点**
2. 找到链表**倒数第n个**节点
3. 判断和检测链表**环(Cycle)**的问题
4. 找到两个链表的**交点**

看题：
[876.Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/description/)
>Given a non-empty, singly linked list with head node head, return a middle node of linked list.
If there are two middle nodes, return the second middle node.  

Example 1:

>Input: [1,2,3,4,5]
Output: Node 3 from this list (Serialization: [3,4,5])
The returned node has value 3.  (The judge's serialization of this node is [3,4,5]).
Note that we returned a ListNode object ans, such that:
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, and ans.next.next.next = NULL.  

Example 2:  
>Input: [1,2,3,4,5,6]
Output: Node 4 from this list (Serialization: [4,5,6])
Since the list has two middle nodes with values 3 and 4, we return the second one.

*思路：*
要找到中点，我们首先想到如果知道链表的长度，那么问题将会很简单，那我们需要先来一趟循环把长度求出来吗？TwoPointers告诉你，不需要，一趟循环解决问题。
```c
void middleList(struct ListNode* head, struct ListNode** mid){
	struct ListNode *fast = head, *slow = head;
	while(fast && fast->next){
		fast = fast->next->next;
		slow = slow->next;
	}
	*mid = slow;
}
```
正确性是显然的。

[19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)
>Given a linked list, remove the n-th node from the end of list and return its head.

Example:

>Given linked list: 1->2->3->4->5, and n = 2.
>After removing the second node from the end, the linked list becomes 1->2->3->5.

同样的我们不需要事先知道链表的长度，我们让快指针先走n步，再让fast and slow并驾齐驱，那么slow和fast的距离**始终**是n, 当fast走到尾节点的时候，slow和尾节点的距离是n，那么就是它了。
  

代码：
```c
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    if(!head)
        return NULL;
    struct ListNode* fast = head;
    struct ListNode* slow = head;
    struct ListNode* tmp = NULL;
    while(n--)
        fast = fast->next;
    if(!fast){
        tmp = head;
        head = head->next;
        free(tmp);
        return head;
    }
    while(fast->next){
        fast = fast->next;
        slow = slow->next;
    }
    tmp = slow->next;
    slow->next = slow->next->next;
    free(tmp);
    return head;
}
```

同样，它对删除头节点的情况作了判断。

[142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/description/)  
>Given a linked list, return the node where the cycle begins. If there is no cycle, return null.

Note: Do not modify the linked list.

思路：设置快慢指针，起点都为head，fast比slow步长多1，那么如果有环存在的话，当slow走到入环位置时，fast已经在环中了，当他们第一次相遇时，fast已经比slow多走了n圈。此时让slow再指向head，和fast以一样的步长来走，当它们再次相遇的位置即为入环位置。

**入环位置**与head的距离：a
环的长度：r
链表的长度 ：L
**第一次相遇**和head的距离： s
**第一次相遇**和入环位置的距离：x

存在以下关系：
s = 2s + nr 
r = L - a
a + x = s = nr = (n-1)r + r = (n-1)r + L - a
a = (n-1)r + (L - a - x)   

最后一式说明让slow从头走，fast从当前位置走，步长为1，同时出发，它们一定会在a位置相遇，也就是入环位置，算法的正确性得以验证。



代码：
```c
struct ListNode *detectCycle(struct ListNode *head) {
    struct ListNode* fast = head;
    struct ListNode* slow = head;
    while(fast != NULL && fast->next != NULL){
        fast = fast->next->next;
        slow = slow->next;
        if(fast == slow){
            slow = head;
            while(fast && slow != fast){
                fast = fast->next;
                slow = slow->next;
            }
            return slow;   
        }
    }
    return NULL;
}
```
[160. Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/description/)
>Write a program to find the node at which the intersection of two singly linked lists begins.


For example, the following two linked lists:

>A:          a1 → a2→ c1 → c2 → c3                  
>B:     b1 → b2 → b3→ c1 → c2 → c3  

begin to intersect at node c1.

这就属于另类快慢指针的运用了，你应该弄清楚它们的指向是如何改变的。经验证，该算法的时间复杂度为O(m+n)。

代码：  
```c
struct ListNode *getIntersectionNode(struct ListNode *headA, struct ListNode *headB) {
    if(headA == NULL || headB == NULL)
        return NULL;
   struct ListNode *pa = headA, *pb = headB;
    while(true){
		if(pa == pb)
			return pa;
		if(pa->next == NULL && pb->next != NULL){
			pa = headB;
			pb = pb->next;
			continue;
		}
		if(pa->next != NULL && pb->next == NULL){
			pb = headA;
			pa = pa->next;
			continue;
		}
		if(pa->next == NULL && pb->next == NULL)
			return NULL;
		pa = pa->next;
		pb = pb->next;		
	}
	return NULL;     
}
```
实际上，`pa`,`pb`的存在仿佛将两个链表连接到了一起，假若有交点，它们必定会相遇，因为实际上它们推进的根本就是**同一条链表**，就是那个**虚拟**链接在一起，长为`m + n`的链表。只是**快慢交错** ，**最坏情况**下 ，它们在m+n个推进后相遇。

更多例子，欢迎访问[我的解答](https://github.com/TuoAiTang/LeetCode)，byebye~



