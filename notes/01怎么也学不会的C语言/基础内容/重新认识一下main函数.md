---
title: 【C语言】重新认识一下main函数
date: 2020-02-14 23:19:00
tags: 
 - C
 - main函数
#categories: 
# - C
# declare: true
---


## 一、main函数返回值

### 1.1 函数为什么需要返回值

>1. 一般而言，参数是函数的输入，返回值是函数的输出(在高级应用中，使用参数放回，放回值说明状态)
>2. 函数需要对外输出数据（函数运行的一些结果值）因此需要返回值
>3. 形式上来说，函数被另一个函数所调用，返回值作为函数式的值返回给调用这个函数的地方
<!--more-->
### 1.2 main函数的正确书写

以下三种方式是正确规范的

```cpp
int main(void){}
int main(int argc, char *argv[]){}
int main(int argc, char **argv){}
```

以下书写方式是及其不规范的！

```cpp
void main(){}//极其不规范！！！
```

### 1.3 main函数被谁调用

* C语言规定`main`函数是整个程序的入口。其他的函数只有直接或间接被main函数所调用才能被执行，从某种角度来讲`main`函数代表了当前的整个程序。main函数的开始意味着整个程序开始执行，`main`函数的结束返回意味着整个程序的结束。可以说，谁执行了这个程序谁就调用了`main`函数！

>1. 表面来看，`linux`中在命令行中去`./xx`执行一个可执行程序
>2. 还可以通过`shell`脚本来调用执行一个程序
>3. 还可以在程序中去调用执行一个程序（`fork exec`）

* 我们有多种方法都可以执行一个程序，但是本质上是相同的。`linux`中一个新程序的执行本质上是一个进程的创建、加载、运行、消亡。`linux`中执行一个程序其实就是创建一个新进程然后把这个程序丢进这个进程中去执行直到结束。命令行本身就是一个进程，在命令行底下`./xx`执行一个程序，其实这个新程序是作为命令行进程的一个字进程去执行的。

### 1.4 mian函数的放回值

* `main`函数返回给调用这个函数的父进程。父进程调用子进程来执行一个任务，字进程执行完后通过`main`函数的返回值返回给父进程一个状态值。这个状态值一般表示子进程是够成功执行。（0表示执行成功，负数表示失败）

* 可以用`shell`脚本执行程序并获取、答应程序的返回值，如下所示

```cpp
int main(void)
{
    return 10;//测试用
}
```

对应的`shell`脚本为

```javascript
#!/bin/sh

gcc main.c -o main.elf
./main.elf
echo $?
```

![clipboard_20200214115058.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200214115058.png)

## 二、argc、argv与main函数的传参

* `main`函数不传参是可以的，也就是说父进程调用子程序并且给子程序传参不是必须的。`int main(void)`这种形式就表示程序员认为不必要给main传参
* 有时候我们希望程序有一种灵活性，选择在执行程序时通过传参来控制程序中的运行，达到不需要重新编译程序就可以改变程序运行结果的效果

### 2.1 argc和argv

`argc`和`argv`这两个C语言预订的参数可以实现给`main`函数传参：

>1. `argc`是`int`类型，表示运行程序的时候给`main`函数传递了几个参数
>2. `argv`是一个字符串数组，这个数组用来存储`argc`个字符串，每个字符串就是我们给`main`函数传的一个参数

### 2.2 给main传参的本质

程序调用虽然有很多方法，但是本质上都是父进程`fork`一个子进程，然后字进程和`exec`函数族绑定起来去执行，我们在`exec`的时候可以给函数传参。二程序调用时可以被传参（也就是`main`的传参）是操作系统层面的支持完成的。在给`main`传参要注意:

>1. `main`函数传参都是通过字符串传进去的
>2. 程序被调用时传参，各个参数之间是通过空格来间隔的
>3. 在程序内部如果要使用`argv`，那么一定要先检验`argc`

### 2.3 测试代码

```cpp
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[])
{
    printf("共传入%d个参数\n", argc);
    for(int i = 0; i < argc; i ++)
    {
        printf("传入的第%d个参数为:%s\n", i + 1, argv[i]);
    }
    if(! strcmp(argv[argc - 1], "thride"))
    {
        printf("Hello World!\n");
    }
    return 0;
}
```

shell脚本为

```javascript
#!/bin/sh

gcc main.c -o main.elf
./main.elf first second thride
echo $?
```

测试结果如下：
![clipboard_20200215122328.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200215122328.png)

## 三、10以内简单计算器的实现

```cpp
#include <stdio.h>
#include <string.h>

static inline int add(int a, int b)
{
    return a + b;
}
static inline int sub(int a, int b)
{
    return a - b;
}

int main(int argc, char *argv[])
{
    if(! strcmp(argv[2], "+"))
    {
        int value = add(*argv[1] - '0', *argv[3] - '0');
        printf("the value of %s %s %s is %d\n", \
        argv[1], argv[2], argv[3], value);
    }
    else if(! strcmp(argv[2], "-"))
    {
        int value = sub(*argv[1] - '0', *argv[3] - '0');
        printf("the value of %s %s %s is %d\n", \
        argv[1], argv[2], argv[3], value);
    }
    else
    {
        printf("ERROR\n");
        return -1;
    }
    return 0;
}
```

![clipboard_20200215124117.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200215124117.png)
