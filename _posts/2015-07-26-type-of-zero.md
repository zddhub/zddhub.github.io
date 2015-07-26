---
layout: post
title: "零值强制类型转换的使用"
category: Knowledge
tags: "c/c++ (type*)0"
---

> 这可能是最最基础的内容了，可是我却从来没在项目中用过。有一句话是对的，永远不能说精通哪一门语言。

零值可被强制转化为任意类型，转化出的结果，不能被直接访问，可获取相应域的偏移量，通过对应域推算外层类型的地址。

<!-- more -->

(unsigned long)&(((type*)0)->field)用法
--------------------------------------

```c
#include <stdio.h>

struct Test1 {
  int a;
  char b;
  float c;
};

struct Test2
{
  struct Test1 t1;
  int a;
  char b;
  float c;
};

int main(int argc, char **argv) {
  struct Test1 t1 = {1, 'z', 1.0};
  struct Test2 t2 = {t1, 2, 'd', 2.0};

  int *pa = &t2.a;
  // 利用（(type*)0)->field 计算域的偏移量，再利用偏移量求出Test2的首地址，并强制转化获取指针。
  struct Test2 *pt2 = (struct Test2 *)((unsigned long)pa - (unsigned long)&(((struct Test2*)0)->a));
  printf("(%d, %c, %f), %d, %c, %f\n", pt2->t1.a, pt2->t1.b, pt2->t1.c, pt2->a, pt2->b, pt2->c);

  // 如果直接通过sizeof计算偏移的话，可能会由于内存对齐而导致的错误。
  printf("%ld %ld\n", sizeof(t1), sizeof(t2));
  printf("%ld %ld %ld %ld\n", (unsigned long) &t2, (unsigned long) &t2.a, (unsigned long) &t2.b, (unsigned long) &t2.c);
}
```
