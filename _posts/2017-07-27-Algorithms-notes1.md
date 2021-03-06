---
title: 'Notes: Pointer's Pointer'
layout: post
tags:
  - leetcode
  - learning notes
  - C++
category: C/C++
---

A simple expalination on the pointer's pointer technic. Best for linked list element delete.

<!--more-->

## Question

Suppose we have a pointer's pointer P, then what's the difference between `*P = (*P)->next` and `P = &((*p)->next)` ?

To explain this question, we see the below program.

```cpp
#include <iostream>
using namespace std;

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

int main() {
    ListNode *n1 = new ListNode(1);
    ListNode *n2 = new ListNode(2);
    ListNode *n3 = new ListNode(3);
    n1->next = n2;
    n2->next = n3;
    
    // Now, 1 -> 2 -> 3 -> NULL
    //      n1   n2   n3

    ListNode **p = &n1;
    p = &((*p)->next);
    (*p) = (*p)->next;
    p = &((*p)->next);

    // Now, 1 -> 3 -> NULL;
    //      n1   n3

    while (n1) {
        cout << n1->val << endl;
        n1 = n1->next;
    }
    return 0;
}
```

The output of this program is:

```
1
3
```

## Analysis

When the linked list is built, it looks like this in memory:

```
    ----
000 val 
    ----
001 002  
    ----
002 val
    ----
003 003
    ----
004 val
    ----
005 NULL
    ----
006
    ----
    
    
    ...
    
    ----
101 000  n1
    ----
102 002  n2
    ----
103 004  n3
    ----
104 101  p
    ----
105
    ----
106
    ----
```
After the code `p = &((*p)->next);`, the memoery between address 000 - 006 remains, while the memory stores pointers becomeslike this:

```
    ----
101 000  n1
    ----
102 002  n2
    ----
103 004  n3
    ----
104 001  p  (Now p stores the addres of pointer n1->next)
    ----
105
    ----
106
    ----
```

Now p stores the addres of pointer n1->next, so change (*p) will change the value stored in pointer n1->next;

And then after the code `(*p) = (*p)->next;`：

```
    ----
000 val 
    ----
001 004  
    ----
002 val
    ----
003 003
    ----
004 val
    ----
005 NULL
    ----
006
    ----

```

So now, pointer n1->next is pointing to n3.


**Note:**

In this article, we did not deal with the dead addresses. By right, we should use `delete` to release the space.

