# 第七章 链接

## 编译器驱动程序
下面通过一个示例程序来来说明`gcc`背后的执行过程
```C
//mian.c
int sum(int *a, int n);
int array[2] = {1,2};

int main()
{
    int val = sum(array,2);
    return val;
}
```

```C
//sum.c
int sum(int *a, int n)
{
    int i, s=0;

    for(i = 0; i < n; i++) {
        s += a[i];
    }
    return s;
}
```
然后执行`gcc -v -Og -o prog main.c sum.c`, -v(verbose, 啰嗦模式)可以输出详细的信息(内容太长, 略), 
可以看到gcc背后的执行流程,可以看到`gcc`背后其实是调用了一系列的程序(`cc1`,`as`,`collect2`)来完成程序的编译链接。

下面演示不用`gcc`, 而是用`cc1`,`as`,`collect2`来编译链接程序

- /usr/lib/gcc/x86_64-linux-gnu/9/cc1 [省略一堆参数] main.c -Og -o main.s # 预处理和编译阶段
-  as -v --64 -o main.o main.s                                           # 汇编阶段 

- /usr/lib/gcc/x86_64-linux-gnu/9/cc1 [省略一堆参数] sum.c -Og -o sum.s   # 预处理和编译阶段
-  as -v --64 -o sum.o sum.s                                             # 汇编阶段

usr/lib/gcc/x86_64-linux-gnu/9/collect2 [省略系统目标文件和其他参数] sum.o main.o -o prog  # 链接阶段,链接一堆的库

总结上面流程如下:
```
main.c  ->  cc1预处理&编译 ->  main.s  ->  as编译  -> main.o
                                                           \
                                                             -> collect2连接他们及系统目标文件  -> prog
                                                           /
sum.c   ->  cc1预处理&编译 ->  sum.s  ->   as编译  ->   sum.o
```

这里看到的结果跟CSAPP 3ed中文版第465页的有点不一样, csapp上说预处理是用 `cpp` (C pre Preprocessor), 链接使用 `ld`, 下面我们来探究他们有什么不用:

1. 使用`cpp -v main.c -o main.i`后可以看到其中有如下输出:
```
/usr/lib/gcc/x86_64-linux-gnu/9/cc1 -E [省略一堆其他参数] main.c -o main.i
```
可以看出本质上`cpp`也是调用`cc1`并且使用了`-E`参数

2. 使用`strace usr/lib/gcc/x86_64-linux-gnu/9/collect2 [省略系统目标文件和其他参数] sum.o main.o -o prog`(strace 可以跟踪系统调用)后, 可以看到其中有如下输出:
```
write(2, "/usr/bin/ld", 11/usr/bin/ld)             = 11
【内容太多,省略其他】
write(2, " -o", 3 -o)                      = 3
write(2, " prog", 5 prog)                    = 5
write(2, " sum.o", 6 sum.o)                   = 6
write(2, " main.o", 7 main.o)                  = 7
【内容太多,省略其他】
```

可以看到本质上来说, `collect2` 其实也是通过调用`ld`实现的.同时可以参考: [GNU Collect2](https://gcc.gnu.org/onlinedocs/gccint/Collect2.html)

> 拓展, 所以我们直接使用`gcc`编译,然后使用`ld`链接他们, 程序是无法正常运行的, 因为gcc直接编译链接其背后帮我们做了很多工作.
 ```
➜  chapter7 gcc -c main.c sum.c && ld main.o sum.o -o a.out
ld: warning: cannot find entry symbol _start; defaulting to 0000000000401000
➜  chapter7 ./a.out
[1]    3756 segmentation fault  ./a.out
➜  chapter7
```

问题环节:
1. 为什么这个程序会发生`segmentation fault`,你可以解释原因吗?                              # 答,用GDB调试,过程略
2. 如果在上面`mian.c` 的主函数返回之前加上如下的死循环代码`while(1);`,程序可以正常运行吗?     # 答, 是可以正常运行的



## 静态链接
