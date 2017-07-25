---
title: 'Mistakes I made in C++ coding'
layout: post
tags:
  - Mistakes
  - C++
category: Algorithms 
---

We always make the same mistake. So I write them down here.

<!--more-->


## Mistakes

```cpp
// In some compilers, 'a' could be INT_MAX;
// But in some compilers, 'a' is not INT_MAX since the 1 << 32 overflow.
int a = (1 << 32) - 1;
```

