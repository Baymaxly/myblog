---
title: 堆(heap)
date: 2020-02-01 11:38:49
tags: 
 - C
 - 数据结构
 - 内存管理
#categories: 
#declare: true
#Author：卢意
---

## 一、堆内存特点

>1. **操作系统堆管理器管理**：堆管理器是操作系统的一个模块，堆管理内存分配灵活，按需分配
>2. **大块内存**：堆内存管理着总量很大的操作系统内存块，各进程可以按需申请使用，使用完释放
>3. **手动申请、释放**：需要写代码去malloc及free
>4. **脏内存**：堆内存也是反复使用的，而且使用者用完释放前不会清除，因此也是脏的
>5. **临时性**：堆内存只在malloc和free之间属于某个进程，可以访问。在malloc之间个free之后都不能访问，会引发不可预知的错误。
<!--more-->
## 二、堆内存使用范例

### 2.1 使用步骤

>1. 申请并绑定内存
>2. 检验是否成功分配
>3. 初始化内存
>4. 使用内存
>5. 释放内存

### 2.2 代码示例

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    //申请一个容纳100个int类型元素的内存并和p绑定
    int *p = (int *)malloc(100 * sizeof(int));
    //检验是否成功分配
    if(!p)
    {
        printf("malloc error.\n");
        return -1;
    }
    //初始化
    memset(p, 0, 100);
    //使用内存
    *(p + 9) = 5;
    for(int i = 0; i < 100; i ++)
    {
        printf((i + 1) % 10 ? "%d " : "%d\n", *(p + i));
    }
    //释放内存
    free(p);
    return 0;
}
```

### 2.3 结果示例

![1GJiXF.png](https://s2.ax1x.com/2020/02/01/1GJiXF.png)

### 2.4 相关说明

>1. `void`类型不是空型，也不是没有类型，表示**万能类型**。表示这个数据的类型当前是不确定的，在需要的时候可以再去指定它的具体类型。
>2. `void *`类型是一个指针类型，这个指针本身只占四个字节，但是指针指向的类型是不确定的，在需要的时候可以再去指定它的具体类型。
>3. `malloc`返回的是一个`void *`类型的指针：内存申请成功后返回的是堆管理器分配给本次申请的那段内存空间的首地址（`malloc`返回的值其实就是一个数字，这个数字表示一个内存地址），申请失败时返回`NULL`
>4. `malloc`使用`void *`作为返回类型是因为`malloc`分配内存时只是分配一个内存空间给进程使用，而并不关心这段内存空间时用来储存什么类型的元素，具体的类型是由程序员来强制转化的。

## 三、内存泄露

当前进程申请内存后：

>1. 在没有`free`前就丢失了这块内存的地址
>2. 退出进程没有`free`这块内存

本进程无法使用或不再使用这块内存，但是操作系统的堆管理器认为该段内存是当前进程拿着的，其他进程也无法使用，就造成了内存泄露，直到当前进程结束时操作系统才会回收这段内存

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    //申请一个容纳100个int类型元素的内存并和p绑定
    int *p = (int *)malloc(100 * sizeof(int));
    //检验是否成功分配
    if(!p)
    {
        printf("malloc error.\n");
        return -1;
    }
    //初始化
    memset(p, 0, 100);
    /*
        使用内存
    */
    p = NULL;//释放前丢失指向导致内存泄露
    //未释放内存导致内存泄露
    //free(p);
    return 0;
}
```

## 四、注意事项

1. `free`之后当前进程就失去了访问那块内存空间的权限，再次访问会引发段错误。
2. 为了避免内存泄露，在调用`free`归还这段内存之前，指向这段内存的指针一定不能丢（不能赋值使其指向另外的地方）。`p`一但丢失这段`malloc`来的内存，就会造成内存泄露
3. `malloc`之后一定要检验内存是否分配成功
4. `free`之后一定要让`p`指向`NULL`，以避免野指针的出现
5. 不用越界访问

## 五、一些没多大实际意义但是有趣的东西

### 5.1 malloc(0)

在硬件电路中0欧姆的电阻确实有很多妙用，但是在软件中`malloc`一个`0字节`的内存就真的是一件很无厘头的事情。

```cpp
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int *p1 = (int *)malloc(0);
    int *p2 = (int *)malloc(0);
    if(!p1 && !p2)
    {
        printf("malloc error!\n");
        return -1;
    }
    printf("p1 = %p\n", p1);
    printf("p2 = %p\n", p2);
    /*记得释放内存并让p指向NULL*/
    return 0;
}
```

![malloc0.png](https://s2.ax1x.com/2020/02/01/1GQ301.png)

根据测试结果我们可以看到`malloc`返回了一个有效指针。其实`C`语言并没有明确规定`malloc(0)`时的具体表现，是由各`malloc`函数库的实现着来定义的，在工程领域不必深究这个东西。

### 5.2 malloc(4)

大家猜一猜这是分配几个`Byte`的内存空间呢？`4Byte`？

```cpp
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int *p1 = (int *)malloc(4);
    int *p2 = (int *)malloc(4);
    int *p3 = (int *)malloc(4);
    if(!p1 && !p2 && !p3)
    {
        printf("malloc error!\n");
        return -1;
    }
    printf("p1 = %p\n", p1);
    printf("p2 = %p\n", p2);
    printf("p3 = %p\n", p3);
    /*记得释放内存并让p指向NULL*/
    return 0;
}
```

![malloc4.png](https://s2.ax1x.com/2020/02/01/1Gla5V.png)

我们可以看到地址之间的差值其实是`32B`或`16B`，其实`malloc`实现时并没有实现任意大小内存的分配，而是成块分配的，最小为`16`字节。也就是说`malloc`小于`16B`的大小时都会向`16B`补齐。

### 5.3 访问并使用超出申请的内存

```cpp
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int *p1 = (int *)malloc(4 * sizeof(int));
    if(!p1)
    {
        printf("malloc error!\n");
        return -1;
    }
    *(p1 + 1) = 1;
    *(p1 + 100) = 100;
    *(p1 + 10000) = 10000;
    for(int i = 1; i < 100000; i *= 100)
    {
        printf("*(p1 + %d) = %d\n", i, *(p1 + i));
    }
    *(p1 + 1000000) = 10000;
    printf("%d", *(p1 + 1000000));
    free(p1);
    p1 = NULL;
    return 0;
}
```

![heapover1.png](https://s2.ax1x.com/2020/02/01/1G3etP.png)

其实，在一定范围内，堆管理器是允许进程对申请的内存进行越界访问的。但是，这种过于开放的管理方式其实有很大的弊端，可能一不小心就会修改掉本进程的其他有效数据（如下所示）或其他进程的数据，所以在正常的场合下是不会越界访问的。好在这有一定的限度，无限向外访问会触发段错误

```cpp
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    int *p1 = (int *)malloc(4 * sizeof(int));
    int *p2 = (int *)malloc(4 * sizeof(int));
    if(!p1 && !p2)
    {
        printf("malloc error!\n");
        return -1;
    }
    *(p2 + 0) = 5;
    *(p1 + 8) = 10; //越界访问
    printf("*(p2 + 1) = %d\n", *(p2 + 0));
    printf("*(p1 + 8) = %d\n", *(p1 + 8));
    free(p1);
    p1 = NULL;
    free(p2);
    p2 = NULL;
    return 0;
}
```

![1G8NVA.png](https://s2.ax1x.com/2020/02/01/1G8NVA.png)

通过测试结果我们可以看出`p1`越界之后把`*(p2 + 0)`的值给修改了！这是极其危险的行为！越塔强杀会有报应的！
