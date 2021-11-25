---
title: 【C语言】宏定义与预处理
date: 2020-02-09 20:07:49
tags: 
 - C
 - 宏定义
 - 预处理
#categories: 
# - C
# declare: true
# author：luyi
---
<!--more-->
## 一、由源码到可执行程序的过程

**1. 预处理：** 源码经过预处理器的预处理变成预处理过的`.i`中间文件

```cpp
gcc -E test.c -o test.i
```

**2. 编译：** 中间文件经过编译器编译形成`.s`的汇编文件

```cpp
gcc -S test.i -o test.s
```

**3. 汇编：** 汇编文件经过汇编器生成目标文件`.o`(机器语言)

```cpp
gcc -c test.s -o test.o
```

**4. 链接：** 链接器将目标文件链接成`.exe`可执行程序(`Linux`下是`.elf`)

```cpp
gcc test.o -o test.exe
```

在整个过程中，预处理用预处理器，编译用编译器，汇编用汇编器，链接用链接器，这几个工具再加上其他额外的可能会用到的工具，合起来叫**编译工具链**。`gcc/g++`就是一个编译工具链，在实际工程中并不会去手动生成那么多中间文件，而是直接一步到位：

```cpp
gcc test.c -o test.exe
```

其中，编译器的主要目的是编译源代码，即将`.c`的源代码（`.i`本质上就是预处理过的`.c`)转化成`.s`汇编代码。为了让编译器聚焦核心功能，就将一些非核心功能剥离到预处理器去了，也就是所让预处理器帮编译器做一些编译前的准备工作。

## 二、编程中常见的预处理

>1. 头文件包含：`#include`
>2. 注释
>3. 条件编译：`#if #elif #endif...`
>4. 宏定义

### 2.1 头文件包含

#### 2.1.1 `#include <>`和 `#include ""`的区别

>* `#include <>`专门用来包含系统提供的头文件，如果使用`<>`，编译器只会到系统指定目录(编译器中配置的或`OS`配置的目录，如在`Ubuntu`中是`/usr/include`，编译器还允许用`-I`来附加指定其他的包含路径)去寻找这个头文件，如果找不到就会提示这个头文件不存在。
>* `#include ""`用来包含程序员写的头文件，编译器默认先在当前目录下寻找相应的头文件，如果没找到再到系统指定目录去寻找，如果还没找到则提示文件不存在。

##### 使用原则

>* 系统自带的用` <>`
>* 程序员写的放在当前目录下用` ""`
>* 程序员写的集中放专门存放头文件的目录下，在编译器中用`-I`参数寻找用` <>`

#### 2.1.2 头文件包含在预处理时的处理方式

在预处理的时候，预处理器将所包含的头文件的内容原处展开替换这`#include`语句。
如下所示分别是同一目录下的`.c`文件和`.h`文件：

![test1.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test1.png)

对其进行预编译生成`.i`文件后，`.i`文件的内容如下所示：

![test2.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test2.png)

### 2.2 注释

我们在`.c`源文件中写的注释，预处理器在预处理阶段会将其擦除(在`.c`文件依然存在，在`.i`文件中不存在)。其实这也正顺应了注释是写给使用程序的人看的，而不是给编译器看的。如下所示：

![test.3.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test.3.png)

### 2.3 条件编译

一般情况下，源程序中所有行都参与编译，但有时希望程序有多种配置，对一部分内容指定编译条件(如产品的调试版与正式版)，这就是**条件编译**`(conditional compile)`，条件编译有以下两种判定方法：

#### 2.3.1 #if

这种判定方法类似于`if...else...`语句，格式为`#if (条件表达式)`，它的判定标准是()中的表达式是`true`还是`flase`，在进行预编译的过程中，只会保留条件表达式为真的那部分内容，如下所示测试代码：

```cpp
#include <stdio.h>
#define DEBUG 1

int main(void)
{
    #if(DEBUG == 1)
    printf("Version: DEBUG!");
    #elif(DEBUG == 2)
    printf("Version: TEST!");
    #else
    printf("Version: LAST!");
    #endif

    return 0;
}
```

预编译结果如下所示：

![test4.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test4.png)

我们也可以不在源码中对`DEBUG`进行宏定义，而在编译的时候可以用如下方法对其进行宏定义并指定值：

```cpp
gcc -E -D DEBUG=2 test.c -o test.i
```

结果如下所示：

![test5.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test5.png)

#### 2.3.2 #ifdef

用`#ifdef XXX`判定条件成立与否时主要是看`XXX`这个符号在本语句之前有没有被定义，只要定义了，判断就成立，并不关心`XXX`的宏值为多少。测试代码及结果参见下文**3.1非带参宏**的内容

## 三、宏定义

### 3.1 非带参宏

非带参宏主要结合条件编译使用，比较简单，其定义格式为

```cpp
#define 宏名 替换列表(替换列表可有可无)
```

如下所示：

```cpp
#define DEBUG
#define TEST 1
#define TEST2 TEST
```

#### 宏定义的预处理

>1. 在预处理阶段由预处理器进行**机械替换**，而不做类型检查
>2. 宏定义替换会递归进行，直到替换出来的值本身不再是一个宏为止

如下是在`STM32`开发过程中常用的打印调试信息的一个调试代码段：

```cpp
#include <stdio.h>

#define USER_TIMER_DEBUG
#ifdef USER_TIMER_DEBUG
#define user_timer_printf(format, ...)  printf( format "\r\n", ##__VA_ARGS__)
#define user_timer_info(format, ...)  printf("[\ttimer]info:" format "\r\n", ##__VA_ARGS__)
#define user_timer_debug(format, ...) printf("[\ttimer]debug:" format "\r\n", ##__VA_ARGS__)
#define user_timer_error(format, ...) printf("[\ttimer]error:" format "\r\n",##__VA_ARGS__)
#else
#define user_timer_printf(format, ...)
#define user_timer_info(format, ...)
#define user_timer_debug(format, ...)
#define user_timer_error(format, ...)
#endif
```

预编译结果如图所示：

![test6.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test6.png)

### 3.2 带参宏

宏可以带参数，称为带参宏。带参宏的使用和带参函数非常相似，只是在使用上和处理上有一些差异，其定义格式为：

```cpp
#define 标识符(参数1,参数2,...,参数n) 替换列表
```

在定义带参宏时，**每一个参数在宏体中引用时都必须加括号，最后整体再加括号，括号缺一不可**。

不带括号的后果：

```cpp
#include <stdio.h>
#define M(a, b) a * b
int main(void)
{
    int result = M(2 + 3, 5)
    printf("%d", result);
    return 0;
}
```

如上测试代码，我们想得到`(2 + 3) * 5`的结果，但是由于宏在预处理的时候也是进行机械替换，`int result = M(2 + 3, 5)`变成了`int result = 2 + 3 * 5`，这及其容易出现逻辑上的错误

#### 3.2.1 带参宏示例

**1.MAX宏:** 求2个数中较大的一个

```cpp
#define MAX(a, b) (((a)>(b)) ? (a) : (b))
```

**2.SEC_PER_YEAR宏** 用宏定义表示一年中有多少秒

```cpp
#define SEC_PER_YEAR (365*24*60*60UL)
```

这个宏需要注意的是

>1. 当一个数字直接出现在程序中时，它的是类型默认是`int`
>2. 一年有多少秒，这个数字超过了`int`类型存储的范围
>3. 加`UL`将其转为无符长整型

#### 3.2.2 带参宏和带参函数的区别

##### 1.时间与空间

>* 宏定义在预处理期间处理，进行简单的内容替换，无需额外空间
>* 函数是在编译期间处理的，调用时需要为形参分配空间并将实参的值赋给形参

##### 2.执行速度

>* 宏只进行文本替换，函数运行阶段参数需要进行出入栈的操作，速度比宏慢

##### 3.类型检查

>* 宏定义不会检查参数的类型，返回值也不会附带类型
>* 而数有明确的参数类型和返回值类型。当调用函数时编译器参数的静态类型检查，如果编译器发现实际传参和参数声明不同时会报警告或错误。

##### 综合比较

宏和函数各有千秋，最大的特点是：用函数的时候程序员不太用操心类型不匹配因为编译器会检查，用宏的时候程序员必须注意实际传参和宏所希望的参数类型一致，或者自行加入类型检查，否则可能编译不报错但是运行有误。

如对MAX宏加入类型检查：

```cpp
#define MAX(a, b) ({\
typeof(a) _a = (a); \
typeof(b) _b = (b); \
(void) (&_a == &_b);\
_a > _b ? _a : _b;})
```

测试代码:

```cpp
#include <stdio.h>

#define MAX(a, b) ({\
typeof(a) _a = (a); \
typeof(b) _b = (b); \
(void) (&_a == &_b);\
_a > _b ? _a : _b;})

#define MAX2(a, b) a > b ? a : b

int main(void)
{
    int a = 2;
    float b = 3.1;
    int result2 = MAX2(a, b);
    typeof(MAX(a, b)) result = MAX(a, b);
    printf("result = %f\n", result);
    printf("result2 = %d\n", result2);
    return 0;
}
```

![test7.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/%E5%AE%8F%E4%B8%8E%E9%A2%84%E5%A4%84%E7%90%86/test7.png)

## 四、内联函数

内联函数本质上是函数，通过在函数定义前加`inline`关键字实现，是编译器负责处理的，可以做参数的静态类型检查。同时也有带参宏的展开特性，运行时没有调用开销。

### 4.1 与常规函数对比

>* 当函数体很短的时候，使用常规函数会造成很大的调用开销,内联函数采用原地展开的方式，没有调用开销
>* 当函数体长的时候，由于内联函数展开会降低寻址效率，所以长函数体不会使用内联函数
>* 内联函数本质上是函数，函数的性质内联函数都有

### 4.2 与宏的对比

>* 参数类型检查。编译过程中，编译器仍可以对其内联函数进行参数检查
>* 便于调试。函数支持的调试功内联函数同样支持，而宏不支持
>* 接口封装。有些内联函数可以用来封装一个接口，而宏不具备这个特性

### 4.3 noinline和always_inline

当函数的函数体很小，而且被大量频繁调用，应该做内联展开时，就可以使用内联函数。但编译器会不会作内联展开，编译器也会有自己的权衡（不合理的内联函数会降低`CPU`寻址效率、函数运行效率、降低代码的可移植性...）。如果想告诉编译器一定要展开，或者不作展开，就可以使用`noinline`或`always_inline`对函数作一个属性声明。

```cpp
static inline int func(int *, int);//编译器权衡是否内联展开
static inline __attribute__((noinline)) int func(int *, int);//不内联展开
static inline __attribute__((always_inline)) int func(int *, int);//内联展开
```

用`static`修饰呢是因为内联函数不一定会内联展开，当多个文件都包含同一个内联函数的定义时，如果没有`static`将函数的作用域限制在各自本地文件内，编译时就有可能报重定义错误
