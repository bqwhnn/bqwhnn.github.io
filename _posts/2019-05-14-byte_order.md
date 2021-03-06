---
layout: post
title: "理解字节序--大端和小端"
description:
categories: [计算机系统]
tags: [computer]
redirect_from:
  -- /2019/05/14
---

* Kramdown table of contents
{:toc .toc}

# 字节序

## 大端和小端

计算机硬件有两种储存数据的方式：大端字节序（big endian）和小端字节序（little endian）。

数值 **0x2211** 使用两个字节储存：高位字节是0x22，低位字节是0x11。

* 大端字节序：高位字节在前，低位字节在后，这是人类读写数值的方法。
* 小端字节序：低位字节在前，高位字节在后，即以 0x1122 形式储存。

**0x1234567** 的大端字节序和小端字节序的写法如下图:

![字节序](https://github.com/bqwhnn/bqwhnn.github.io/blob/master/resourses/ByteOrder_01.png?raw=true)

## 为什么会有小端字节序

答案是，计算机电路先处理低位字节，效率比较高，因为计算都是从低位开始的。所以，计算机的内部处理都是小端字节序。

但是，人类还是习惯读写大端字节序。所以，除了计算机的内部处理，其他的场合几乎都是大端字节序，比如网络传输和文件储存。

## 字节序处理

> "只有读取的时候，才必须区分字节序，其他情况都不用考虑。"

处理器读取外部数据的时候，必须知道数据的字节序，将其转成正确的值。

如果是大端字节序，先读到的就是高位字节，后读到的就是低位字节。小端字节序正好相反。

然后，就正常使用这个值，完全不用再考虑字节序。

即使是向外部设备写入数据，也不用考虑字节序，正常写入一个值即可。外部设备会自己处理字节序的问题。

# 字节序测试程序

不同 cpu 平台上字节序通常也不一样，下面的 C 程序可以测试不同平台上的字节序。

``` c
#include <stdio.h>
#include <netinet/in.h>

int 
main() {
    int n = 0x12345678;

    // 取 n 的地址并强制转换为 char * 类型的指针，每次执行 +1 运算内存偏移 8bit
    // 一个 16 进制数占据 4bit 内存，所以一次打印两位 16 进制数
    printf("[0]: 0x%x\n", *((char *)&n + 0));
    printf("[1]: 0x%x\n", *((char *)&n + 1));
    printf("[2]: 0x%x\n", *((char *)&n + 2));
    printf("[3]: 0x%x\n", *((char *)&n + 3));

    n = htonl(n);
    printf("[0]: 0x%x\n", *((char *)&n + 0));
    printf("[1]: 0x%x\n", *((char *)&n + 1));
    printf("[2]: 0x%x\n", *((char *)&n + 2));
    printf("[3]: 0x%x\n", *((char *)&n + 3));

    return 0;
}
```

在80X86CPU平台上，执行该程序得到如下结果：   
```
[0]:0x78  
[1]:0x56   
[2]:0x34   
[3]:0x12  

[0]:0x12   
[1]:0x34   
[2]:0x56   
[3]:0x78  
```

分析结果，在80X86平台上，系统将多字节中的低位存储在变量起始地址，使用小端法。htonl将i_num转换成网络字节序，可见网络字节序是大端法。

# 参考链接

* [理解字节序](http://www.ruanyifeng.com/blog/2016/11/byte-order.html)
* [C语言中大端字节序与小端字节序的转化](https://blog.csdn.net/yishengzhiai005/article/details/39672529)