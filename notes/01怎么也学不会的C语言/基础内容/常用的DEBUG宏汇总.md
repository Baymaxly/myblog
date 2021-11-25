## 一、`printf`函数与`fprintf`函数

### 1.1 `printf`

`printf`的函数原型为：

```cpp
    int printf(const char *format, ...);
```

用于将格式化后的字符串输出到标准输出。标准输出，即标准输出文件，对应终端的屏幕。`printf`称为可变参数函数，其定义声明和普通函数相同。在其参数列表中，`format`称为固定参数部分，`...`称为参数占位符，二者共同构成可变参数。也就是说，它除了有一个参数`format`固定以外，后面的参数其个数和类型都是可变的。

### 1.2 `fprintf`

`fprintf`的函数原型为：

```cpp
    int fprintf(FILE *stream, const char *format, ...);
```

用于把格式化字符串输出到指定文件中，所以参数列表比`printf`多了个文件指针`FILE *`,那是目标文件的文件描述符(文件流指针)，`stdout`即标准输出文件，对应屏幕的终端。也就是说`fprintf`的第一个参数为`stdout`时，等效于`printf`

## 二、几个预定义宏

预定义宏|在预编译时被替换成的内容
:-|:-
 `__FILE__`|当前的源文件名
 `__LINE__`|当前的行号
 `__FUNCTION__`|当前的函数名称
 `__DATE__`|当前的编译日期
 `__TIME__`|当前的编译时间

## 三、两个常用debug宏

### 3.1 普通版

```cpp
#define USER_TIMER_DEBUG
#ifdef USER_TIMER_DEBUG
#define user_timer_debug(format, ...) printf(\
"[\ttimer]debug:" format "\r\n", ##__VA_ARGS__)
#else
#define user_timer_debug(format, ...)
#endif
```

说明：

1. `printf`中的`format`和宏体中的`format`对应，会进行直接的替换
2. `__VA_ARGS__`是一个可变参数宏，在预编译阶段会替代宏体中`...`的内容
3. `__VA_ARGS__`宏前面加上`##`的作用在于，当可变参数的个数为`0`时，把前面多余的`","`除去，否则会编译出错，如下图所示：
![clipboard_20200214103712.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200214103712.png)
加上##则不会出错：
![clipboard_20200214103839.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200214103839.png)

### 3.2 进阶版

```cpp
#define DEBUG
#ifdef DEBUG
#define DBG(format, ...) fprintf(stdout, \
"[\tDBG](File:%s, Func:%s(), Line:%d): " \
, __FILE__, __FUNCTION__, __LINE__);     \
fprintf(stdout, format"\r\n", ##__VA_ARGS__)
#else
#define DBG(format, ...)  do {} while (0)
#endif
```

此版本可以自行打印调试的文件信息，加上`do {} while (0)`是为了消除`;`，不消除也不会引起语法错误

在`gcc`中还支持在宏体中用`args...`代替`...`，表示后续的`args`可能会有多个，在函数中则用`##args`代替`##__VA_ARGS__`，如下所示

```cpp
#define LOG
#ifdef LOG
#define LG(format, args...) fprintf(stdout,\
"[\tLOG](File:%s, Func:%s(), Line:%d): "   \
, __FILE__, __FUNCTION__, __LINE__);       \
fprintf(stdout, format"\r\n", ##args)
#else
#define LG(format, args...)  do {} while (0)
#endif
```

### 3.3 测试

```cpp
#include "debug.h"

void func(void);

int main(void)
{
    int a = 10;
    user_timer_debug("the value of a is %d\n", a);
    DBG("the value of a is %d\n", a);
    func();
    return 0;
}

void func(void)
{
    int b = 100;
    LG("the value of b is %d\n", b);
}
```

![clipboard_20200214110012.png](https://graph-1301143676.cos.ap-chengdu.myqcloud.com/C%E9%AB%98%E7%BA%A7/debug/clipboard_20200214110012.png)