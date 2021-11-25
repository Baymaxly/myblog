---
title: typedef的应用
date: 2020-01-27 22:31:49
tags: 
 - C
 - typedef
#categories: 
#declare: true
#Author：卢意
---

# typedef的应用

![1uhFAS.png](https://s2.ax1x.com/2020/01/27/1uhFAS.png)
<!--more-->
## 一、与typedef相关的预备知识

### 1.1 C语言的两种类型

>1. **内建类型`ADT`**（也叫原生类型、基础数据类型）：编译器自带的类型，如`int/double`等
>2. **自定义类型`UDT`**：不是编译器自带的类型，是程序员自己定义的，如结构体类型、数组类型、函数指针类型等

### 1.2 typedef加工的是类型而不是变量

* 类型就是一个数据模板，变量是一个实在的数据
* 类型是不占内存的，变量是占内存的
* 在面向对象的语言中，类型就是类`class`，变量就是对象
* `typedef`不新建类型，只是对现有的类型进行重命名

## 二、`typedef`与`#define宏`的区别

### 2.1 原理不同

* `typedef`是关键字，它在自己的作用域内给一个已经存在的类型一个别名，在编译时处理，有类型检查功能。
* `#define`是`C语言`中的宏定义，是预处理指令，在预处理时进行简单而机械的字符串替换，不作正确性检查

### 2.2 功能不同

* `typedef`用来定义类型的别名，方便使用
* `#define`除为类型重命名外，还可以定义常量、变量、简化复杂运算

### 2.3 对指针的操纵范围不同

#### 2.3.1 测试代码

```cpp
typedef char* pChar1;
#define pChar2 char*

int main(void)
{
    pChar1 c1, c2;
    pChar2 c3, c4;
    char* p;
    p = c1;
    p = c2;
    p = c3;
    p = c4;
    return 0;
}
```

#### 2.3.2 测试结果

![1u8hDK.png](https://s2.ax1x.com/2020/01/27/1u8hDK.png)

我们可以看到，在`p = c4`处有警告：

`warning: assignment makes pointer from integer without a cast`

即:`警告：在没有强制转换的情况下将整数值赋给指针`

也就是说`c4`其实只是一个`char`型变量而`c2`不是！！！

## 三、`typedef`与结构体

### 3.1 结构体在使用时的常规步骤：

1. 先定义结构体的类型
2. 结构体的类型去定义变量
3. 使用结构体成员

```cpp
/*定义结构体类型，不占内存*/
struct student
{
    char name[20];
    int age;
    char sex[10];
};

int main(void)
{
    struct student s1;
    //struct student是类型，不占内存；s1是变量，占内存
    s1.age = 20;//结构体成员的使用
}
```

### 3.2 typedef简化变量定义

`C语言`语法规定，结构体使用时必须是`struct 结构体类型名 结构体变量名`的形式，如果需要大量使用结构体，这样略为麻烦，可以使用`typedef`来简化，如：

```cpp
typedef struct student
{
    char name[20];
    int age;
    char sex[10];
}student_t;
```

这时就一下定义了两个类型名：`struct student`和`student_t`；
也就是说`struct student s1;`就可以简化定义为`student_t s1`

在很多情况下，会直接简化为：

```cpp
typedef struct student
{
    char name[20];
    int age;
    char sex[10];
}student;
```

同时使用两个`student`在`C语言`中是允许的，这也同时定义了两个类型名：
`struct student`和`student`

在结构体的定义时，也可以同时定义两个类型，如下所示：

```cpp
typedef struct student
{
    char name[20];
    int age;
    char sex[10];
}student, *pStudent;
```

在这里面，第一个是结构体类型，有两个类型名：`struct student`和`student`
第二个是结构体指针类型：有两个类型名：`struct student *`和`pStudent`

**测试代码：**

```cpp
int main(void)
{
    student s1;
    s1.age = 20;
    pStudent p1 = &s1;
    printf("student s1 age = %d\n", p1 -> age);
    return 0;
}
```

**测试结果：**

![](https://s2.ax1x.com/2020/01/27/1uBBt0.png)


## 四、`typedef`与`const`

### 4.1 代码段1

```cpp
typedef int* PINT;
const PINT p;
```

相当于是 `int *const p;`
也就是说`p`是不可变的，`p`指向的变量是可变的

### 4.2 代码段2

```cpp
typedef int* PINT;
PINT const p;
```

也相当于是 `int *const p;`

### 4.3 如果想得到`const int *p;`

```cpp
typedef const int *CPINT;
CPINT p;
```

### 4.4 测试代码：

```cpp
#include <stdio.h>

typedef int *PINT;
typedef const int *CPINT;

int main(void)
{
    int arr[2] = {1, 2};
    /*指向的初始地址*/
    const PINT p0 = &arr[0];
    PINT const p1 = &arr[0];
    CPINT p2 = &arr[0];
    /*尝试改变指向的地址*/
    p0 = &arr[1];
    p1 = &arr[1];
    p2 = &arr[1];
    /*尝试改变指向的变量*/
    *p0 = arr[1];
    *p1 = arr[1];
    *p2 = arr[1];

    return 0;
}
```

### 4.5 编译结果：

![1usJ1K.png](https://s2.ax1x.com/2020/01/27/1usJ1K.png)

我们可以看到编译时报了三个`error`:
```
4.5typedef.c:52:8: error: assignment of read-only variable ‘p0’
     p0 = &arr[1];
        ^
4.5typedef.c:53:8: error: assignment of read-only variable ‘p1’
     p1 = &arr[1];
        ^
4.5typedef.c:58:9: error: assignment of read-only location ‘*p2’
     *p2 = arr[1];
         ^
```

即`p0、p1、*p2`都是只读的，也就应证了前面所说的三个代码段。

## 五、`typedef`与函数指针

如`c标准库`中的`strcpy`函数：
``char *strcpy(char *dest, const char *src);``

定义三个指向该函数的函数指针：

```cpp
char * (*p1) (char *, const char *);
char * (*p2) (char *, const char *);
char * (*p3) (char *, const char *);
```

使用typedef来简化：
```cpp
typedef char * (*pFunc) (char *, const char *);
pFunc p1, p2, p3;
```
其中，`typedef`重命名了一个类型，这个类型新名字叫`pFunc`；
类型是：``char * (*) (char *, const char *);``

## 六、使用`typedef`的意义

1. 简化类型的描述
2. 创造平台无关类型
在很多编程体系下，人们倾向于不使用`int、double`等这些与平台相关的平台内建类型（`int`在`32位`机上是`32`位的，`64位`机上是`64位`的）。为了解决这个问题，很多程序使用自定义的中间类型来缓冲。比如：

```cpp
/*自定义中间类型来做缓冲*/
typedef int sizet_32;
typedef int lent_64;
typedef volatile unsigned int vn32;
/*使用自定义类型来定义变量*/
sizet_32 a;
lent_64 b;
vn32 c;
```
