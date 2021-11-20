---
title: pta
date: 2021-11-20 17:57:36
tags:
---
//实现一个简单的链表拼接
void insertion_sort(LinkNode *&L)
{
    //创造一个有序区
    LinkNode *p;
    LinkNode *pre;
    p=L->next->next;
    LinkNode *q;
    L->next->next=NULL;
    pre=L->next;
    LinkNode *z=L;
    while(p)
    {
        q=p->next;
        while(pre&&pre->data<p->data)
        {
         z=pre;
         pre=pre->next;
        }
        p->next=pre;
        z->next=p;
        p=q;
    }
    
}
第二次错误点在有序区的头指针 没有放在while里面更新
