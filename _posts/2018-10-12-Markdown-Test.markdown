---
layout: post
title: "MarkDown Test"
subtitle: MarkDown and Jekyll Test"
author: "Tuoc"
header-img: "img/post-bg-css.jpg"
header-img-credit: "@WebdesignerDepot"
header-img-credit-href: "medium.com/@WebdesignerDepot/poll-should-css-become-more-like-a-programming-language-c74eb26a4270"
header-mask: 0.4
tags:
  - Web
  - CSS
---
# 你好

- 我
- 爱

> here are some votes

## [ME](baidu.com)
![](https://avatars3.githubusercontent.com/u/33890766?s=400&u=baa1a7c33678722e684f6cb116fead9d8d7cc6b3&v=4)

### 你好

1. 标题
2. *斜体*，**粗体**
3. 图片
4. 链接
5. 代码块
6. 单行代码
7. 引用

` print("hello world")`

### 代码块

``` c
#include <stdio.h>
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

struct ListNode* createList(int n){
    struct ListNode* head = NULL, *tail = NULL;
    for (int i = 0; i < n; ++i)
    {
        struct ListNode* p = (struct ListNode*)malloc(sizeof(struct ListNode));
        p->val = i + 1;
        p->next = head;
        tail = tail?tail:p;
        head = p;
    }
    tail->next = head->next->next;
    return head;
}

struct ListNode *detectCycle(struct ListNode *head) {
    struct ListNode* fast = head;
    struct ListNode* slow = head;
    while(fast != NULL && fast->next != NULL){
        fast = fast->next->next;
        slow = slow->next;
        if(fast == slow)
            return slow;
    }
}

void printLL(struct ListNode* obj){
    struct ListNode* p = obj;
    while(p){
        printf("%d->", p->val);
        p = p->next;
    }
    printf("NULL\n");
}

int main(){
    struct ListNode* head = createList(5);

    printLL(head);

    // struct ListNode* p = detectCycle(head);

    // printLL(p);

    return 0;
}

```
