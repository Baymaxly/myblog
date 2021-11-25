---
title: 【C语言】共用体union与大小端模式
date: 2020-02-07 22:36:49
tags: 
 - C
 - 共用体
 - 大小端模式
#categories: 
# - C
# declare: true
# author：luyi
---

## 一、共用体union

> 电子科技协会 卢意 20210813

### 1.1 共用体的定义、变量定义和使用

共用体`union`和结构体`struct`在类型定义、变量定义和使用方法上很相似，如下代码段

<!--more-->

```cpp
#include <stdio.h>
typedef union myunion1
{
    int a;
    char b;
}MU1;

int main(void)
{
    MU1 test1;
    test1.a = 1;
    printf("test1.a = %d\n", test1.a);
    return 0;
}
```

### 1.2 共用体和结构体的不同

>* 结构体中的成员彼此是独立存在的，分布在不同的内存单元中
>* 共用体重的成员是“一体的”，使用同一个内存单元

#### 1.2.1 测试代码

```cpp
#include <stdio.h>

typedef union myunion1
{
    int a;
    char b;
}MU1;

typedef union myunion2
{
    int a;
    char b;
    char c[9];
}MU2;

int main(void)
{
    MU1 union1;
    union1.a = 65;
    //测试大小
    printf("sizeof(MU1) = %lu\n", sizeof(MU1));
    printf("sizeof(MU2) = %lu\n", sizeof(MU2));
    //测试成员变量地址
    printf("&union1    = %p\n", &union1);
    printf("&union1.a  = %p\n", &union1.a);
    printf("&union1.b  = %p\n", &union1.b);
    //测试成员变量的值
    printf("union1.a  = %d\n", union1.a);
    printf("union1.b  = %c\n", union1.b);
    return 0;
}
```

#### 1.2.2 测试结果

![union1.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/union1.png)

#### 1.2.3 结果分析

>* 共用体整体采用的是4字节对齐，里面的成员变量共用一块内存空间，起始地址均为共用体的首地址
>* 共用体的长度由其最长的成员变量结合整体4字节对齐共同决定，如下图所示
>* 我们知道，字符`'A'`的`ASCII`码为`65`，这也印证了共用体`union`就是对同一块内存中存储的二进制的不同解析方式

![UN2.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/UN2.png)

### 1.3 指针方式访问共用体实现不同解析方式

我们知道共用体的所有成员变量的首地址都是在一起的，这省去了计算各成员变量地址的麻烦，在指针访问时，我们只需要结合不同解析方式对指针类型进行变换即可

#### 1.3.1 测试代码

```cpp
#include <stdio.h>

typedef union myunion1
{
    int a;
    char b;
}MU1;

int main(void)
{
    MU1 union1;
    union1.a = 65;
    printf("*((int *) &union1) = %d\n", *((int *) &union1));
    printf("*((char *)&union1) = %c\n", *((char *)&union1));
    return 0;
}
```

#### 1.3.2 测试结果

![UN3.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/UN3.png)

这个也非常好理解，我们只是将`MU1 *`类型的指针强制转换成了我们所期待的指针类型然后再进行解引用。

## 二、大小端模式

### 2.1 大小端模式简述

#### 2.1.1 大小端问题的由来

在通信系统中，如果要通过串口发送一个`12345678H`给接收方，但是串口只能以字节为单位来发送，需要发4次；接收方分4次接收，内容分别是：`12H`、`34H`、`56H`、`78H`。接收方接收到这4个字节之后需要去重组得到`12345678H`（而不是得到`0x78563412`），因而在通信双方需要有一个协议，就是：先发/先接的是高位还是低位？这就是通信中的大小端问题

#### 2.1.2 计算机中存储系统的大小端

C语言中除了有`8bit`的`char`型，也有`16bit`的`short`型，还有`32bit`的`int`型(据具体的编译环境而定），但是在计算机存储系统中数据仍然是按照字节为单位的。于是一个`16/32/64`位的二进制数据在存储时有2种分布方式：

>* `大端模式(big endian)`   :数据的高字节保存在内存的低地址，低字节保存在高地址
>* `小端模式(little endian)`:数据的高字节保存在内存的高地址，低字节保存在低地址

如下图所示，将数据`12345678H`存入内存中：

![endian1.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/endian1.png)

大端模式和小端模式没有优劣之别，由于小端模式将地址的高低和数据位权结合起来了(高地址部分权值高，低地址部分权值低)，导致主流用小端(`Intel、ARM`)，但是也有些CPU公司用大端(如`C51`)，所以要求必须存储时和读取时按照同样的大小端模式来进行，否则会出错。

### 2.2 用函数测试计算机的大小端存储

我们知道，如果将一个`int`型(`32bit`)的数据存入内存中，需要占`4B`的空间，而`char`型的数据只占`1B`。那么，如果我们将`int`类型的数据存入内存中，而用`char`去解析他，只会得到其中的一部分数据。如`2.1.2`中的`12345678H`，如果用`char`去解析，在小端模式下得到的是`78H`，而在大端模式下得到的是`12H`。

那么我们将`int`类型的`1`存入内存中用`char`解析会怎么样呢？

![01H.PNG](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/01H.png)

很显然：

>* 在大端模式下得到的是`00H`，也就是`0`
>* 在小端模式下得到的是`01H`，也就是`1`

#### 2.2.1 共用体实现

```cpp
#include <stdio.h>

enum MyStatus{BIG = 0, LITTLE};

typedef union BIG_OR_LITTLE_ENDIAN
{
    int a;
    char b;
}BOLE;

int endian_test(void);

int main(void)
{
    int result = endian_test();
    printf(result == BIG ? "大端模式\n" : "小端模式\n");
    return 0;
}

int endian_test(void)
{
    BOLE test;
    test.a = 1;
    return test.b;
}
```

在小端模式下的测试结果(`Intel i5-8300H`)

![unionlittle.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/unionlittle.png)

在大端模式下的调试结果(`Keil C51，AT89C52`)，我们可以看到断点处`result`的值是`0`，说明确实处于大端模式下

![unionbig.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/unionbig.png)

#### 2.2.2 指针实现

```cpp
#include <stdio.h>

enum MyStatus{BIG = 0, LITTLE};

int endian_test(void);

int main(void)
{
    int result = endian_test();
    printf(result == BIG ? "大端模式\n" : "小端模式\n");
    return 0;
}

int endian_test(void)
{
    int test = 1;
    return *((char *)&test);
}

```

在小端模式下的测试结果(`Intel i5-8300H`)

![plittle.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/plittle.png)

在大端模式下的调试结果(`Keil C51，AT89C52`)(不知道为啥这次一截图result (D:0X08) = 0X0000就会没用，所以用手机拍的图...)

![pbig.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%85%B1%E7%94%A8%E4%BD%93%E3%80%81%E5%A4%A7%E5%B0%8F%E7%AB%AF%E3%80%82%E6%9E%9A%E4%B8%BE/pbig.jpg)