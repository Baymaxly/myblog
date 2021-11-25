---
title: offsetof宏与container_of宏
date: 2020-02-05 11:30:49
tags: 
 - C
 - 结构体
 - 宏
 - Linux
#categories: 
# - C
# declare: true
# author：luyi
---

通过结构体的整体变量来访问其中的各个元素，形式上是通过`.`/`->`来访问，实质上是通过指针方式来访问的（编译器会自动计算偏移量）。在`C/C++`中，其实可以用一些宏来计算结构体中成员变量与结构体的地址关系，那就是`offsetof宏`和`container_of宏`

<!--more-->
## 一、offsetof宏

`offsetof`宏：通过虚拟一个`type`类型的结构体变量，然后用`type.member`的方式来访问`member`这个成员变量，继而得到`member`相对于整个变量首地址的偏移量

### 1.1 `offsetof`宏的实现

在C标准库中有对`offsetof`的声明，需包含头文件`stddef.h`：

```cpp
size_t offsetof(type, member);
```

在`Linux`内核中的实现原理如下：

```cpp
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```

>* `TYPE`是结构体类型，`MEMBER`是结构体中一个成员变量名
>* 宏返回的是`MEMBER`相对于整个结构体变量的首地址的偏移量，类型是`unsigned int(即size_t)`

### 1.2 解析`&((TYPE *)0)->MEMBER`

注意：优先级 `->` 高于 `&`

>* `(TYPE *)0`是一个强制类型转化，将数字`0`强制类型转换成`TYPE *`类型，此时这里的`0`代表内存地址`0`，指针为`0`，即可以看做结构体首地址为`0`，相当于在此处作为首地址虚拟一个`TYPE`类型的结构体变量
>* `((TYPE *)0)->MEMBER`得到该结构体变量中的`MEMBER`成员变量
>* `&(((TYPE*)0)->MEMBER)` 使用取地址符&取得了`MEMBER`成员变量的地址，又因为整个虚拟结构体的起始地址是`0`，因此返回值在数值上就等于实际结构体中`MEMBER`成员相对于实际结构体首地址的偏移量

#### 测试代码

```cpp
/*测试环境：64位操作系统下的64位编译器*/
#include <stdio.h>

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

typedef struct mystruct1
{
    int a;
    double b;
    char c;
}MS1;

int main(void)
{
    MS1 test1;
    printf("offsetof_a = %lu\n", offsetof(MS1, a));
    printf("offsetof_b = %lu\n", offsetof(MS1, b));
    printf("offsetof_c = %lu\n\n", offsetof(MS1, c));

    printf("a对于结构体的偏移量为：%lu\n", \
        (char *)&test1.a - (char *)&test1);
    printf("b对于结构体的偏移量为：%lu\n", \
        (char *)&test1.b - (char *)&test1);
    printf("c对于结构体的偏移量为：%lu\n", \
        (char *)&test1.c - (char *)&test1);

    return 0;
}
```

#### 测试结果

![1r2vqA.png](https://s2.ax1x.com/2020/02/05/1r2vqA.png)

有了这个之后我们对结构体进行指针式访问就方便多了，如下述代码段：

```cpp
    int    *pa = (int *)   ((char *)&test1 + offsetof(MS1, a));
    double *pb = (double *)((char *)&test1 + offsetof(MS1, b));
    char   *pc = (char *)  ((char *)&test1 + offsetof(MS1, c));
```

### 1.3 使用`offsetof`宏是否会影响`NULL`

如果会的化，这个宏就不会活到现在还能被我们使用了，为什么不会呢？我们来看看下面的测试代码

```cpp
#include <stdio.h>

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

typedef struct mystruct1
{
    int a;
    double b;
    char c;
}MS1;

int main(void)
{
    size_t offset_a = offsetof(MS1, a);
    size_t offset_b = offsetof(MS1, b);
    size_t offset_c = offsetof(MS1, c);
    return 0;
}
```

我们对其进行汇编处理，得到的汇编代码如下：

```cpp
    .file    "offsetof2.c"
    .text
    .globl  main
    .type   main, @function
main:
.LFB0:
    .cfi_startproc
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    movq    $0, -24(%rbp)
    movq    $8, -16(%rbp)
    movq    $16, -8(%rbp)
    movl    $0, %eax
    popq    %rbp
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc
.LFE0:
    .size   main, .-main
    .ident  "GCC: (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0"
    .section    .note.GNU-stack,"",@progbits
```

我们可以看出，在编译阶段编译器就已经计算出了各成员变量对于结构体的偏移量，如下所示。

```cpp
    movq    $0, -24(%rbp)
    movq    $8, -16(%rbp)
    movq    $16, -8(%rbp)
```

内存中的`NULL`在机器指令中根本未被操作，所以不会影响其值。

## 二、`container_of`宏

`container_of`宏，又称为内核第一宏，他的作用是通过一个结构体中某个成员变量的指针来反推这个结构体变量的指针，有了这个宏，我们可以通过一个成员量的地址得到结构体的地址从而得到结构体中任何一个成员变量的地址。

### 2.1 宏的实现

在`Linux`内核中，`container_of`宏的实现原理如下：

```cpp
#define container_of(ptr, type, member) ({              \
    const typeof(((type *)0)->member) * __mptr = (ptr); \
    (type *)((char *)__mptr - offsetof(type, member)); })
```

>* `ptr`是指向结构体成员变量`member`的指针，`type`是结构体类型(不能填变量），`member`是结构体中的一个成员变量名
>* 这个宏返回的就是整个结构体变量的首地址，类型是`(type *)`
>* `typeof`是`GNU`对`C`关键字的一个拓展，用来获取一个变量或表达式的类型。

### 2.2 原理剖析

>1. 在给宏传成员变量的时候，我们只传入了其地址ptr及名称member，将其类型丢失了。由于结构体的成员数据类型可以是任意的数据类型，所以采用`typeof(((type *)0)->member)`让这个宏兼容各种数据类型。
>2. `__mptr`作为一个临时指针变量存储结构体成员`member`的地址`ptr`
>3. `(char *)__mptr - offsetof(type, member)`中，用结构体成员的地址(强转为`char *`是为了消除附加偏移)减去其相对于结构体的偏移量，即得到了结构体的地址。最后强转为`(type *)`类型进行返回。

### 2.3 测试代码及结果

```cpp
#include <stdio.h>

#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
#define container_of(ptr, type, member) ({              \
    const typeof(((type *)0)->member) * __mptr = (ptr); \
    (type *)((char *)__mptr - offsetof(type, member)); })

typedef struct mystruct1
{
    int a;
    double b;
    char c;
}MS1;

int main(void)
{
    MS1 test1;
    printf("test1的指针为：%p\n", &test1);

    MS1 *pms1 = NULL;
    double *pb = &(test1.b);
    pms1 = container_of(pb, MS1, b);
    printf("pms1的值为：   %p\n", pms1);

    return 0;
}
```

![containerof1.png](https://s2.ax1x.com/2020/02/05/1sVKk4.png)
