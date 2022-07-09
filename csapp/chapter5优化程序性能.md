## 编译器优化的能力和局限性

来看下面一个例子:
```C
// gcc -O1 -c twiddle.c && objdump -d ./twiddle.o
void twiddle1(long *xp, long *yp) {
    *xp += *yp;
    *xp += *yp;
    /**
     xp is rdi, yp is rsi
0000000000000000 <twiddle1>:
   0:   f3 0f 1e fa             endbr64 
   4:   48 8b 06                mov    (%rsi),%rax  # rax = *yp
   7:   48 03 07                add    (%rdi),%rax  # rax = *xp + *yp
   a:   48 89 07                mov    %rax,(%rdi)  # *xp = *xp + *yp
   d:   48 03 06                add    (%rsi),%rax  # rax = *yp
  10:   48 89 07                mov    %rax,(%rdi)  # *xp = *xp + *yp
  13:   c3                      retq   
     */
}

void twiddle2(long *xp, long *yp) {
    *xp += 2 * (*yp);
    /**
     * xp is rdi, yp is rsi
0000000000000014 <twiddle2>:
  14:   f3 0f 1e fa             endbr64 
  18:   48 8b 06                mov    (%rsi),%rax # rax = *yp
  1b:   48 01 c0                add    %rax,%rax   # rax = *yp + *yp
  1e:   48 01 07                add    %rax,(%rdi) # *xy = *xp + rax, 即*xp = *xp + 2 * (*yp)
  21:   c3  
     */
}
```

咋一看, 这两个过程似乎有相同的行为, 而且`twiddle2`内存引用少, 执行效率也会高。
不过考虑xp 等于 yp的情况, 此时函数`twiddle1`会执行下面的计算:
```c
    *xp += *yp;
    *xp += *yp;
```
结果xp的值增加4倍。另一方面, 函数`twiddle2`却是xp增加3倍。

> 这就造成一个主要的妨碍优化的因素,这也是可能严重限制编译器产生话有代码机会的程序的一个方面。
> 如果编译器不能确定两个指针是否指向同一个位置, 那就必须假设什么情况都有可能, 这就限制了可能的优化策略


## 程序优化示例

下面再看一个例子, 看我们是怎样一步一步优化这个程序的:
```C
#include <stdio.h>
#include <assert.h>
#include <stdlib.h>

#define IDENT 1
#define OP *

typedef long data_t;

typedef struct vec
{
    long len;
    data_t *data;
} ver_rec, *vec_ptr;

// if fail, return NULL
vec_ptr new_vec(long len)
{
    assert(len >= 0);

    vec_ptr result = (vec_ptr)malloc(sizeof(ver_rec));
    data_t *data = NULL;
    if (!result)
        return NULL;

    result->len = len;

    if (len > 0)
    {
        data = (data_t *)calloc(len, sizeof(data_t));
        if (!data)
        {
            free(result);
            return NULL;
        }
    }

    result->data = data;

    return result;
}

/**
 * Get vector element and store at dest
 * return 0(out of bounds) or 1(successful)
 */
int get_vec_element(vec_ptr v, long index, data_t *dest)
{
    assert(v != NULL && dest != NULL);

    if (index < 0 || index >= v->len) return 0;
    
    *dest = v->data[index];

    return 1;
}

long vec_length(vec_ptr v)
{
    assert(v != NULL);
    return v->len;
}
```

`combine`是计算向量的乘积,我们一步一步将函数优化:
- 未优化版本
```C
/* Implementation with maximum use of data abstraction */
void combine1(vec_ptr v, data_t *dest)
{
    long int i;

    *dest = IDENT;
    for (i = 0; i < vec_length(v); i++) {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }
}
```

- 消除循环的低效率
```C
/* Move call to vec_length out of loop */
void combine2(vec_ptr v, data_t *dest)
{
    long int i;
    long int length = vec_length(v);

    *dest = IDENT;
    for (i = 0; i < length; i++) {
        data_t val;
        get_vec_element(v, i, &val);
        *dest = *dest OP val;
    }
}
```

- 减少函数调用
```C
data_t *get_vec_start(vec_ptr v) {
    assert(v != NULL);
    return v ->data;
}

/* Direct access to vector data */
void combine3(vec_ptr v, data_t *dest)
{
    long int i;
    long int length = vec_length(v);
    data_t *data = get_vec_start(v);//破坏了原来程序的抽象和封装(个人不建议)

    *dest = IDENT;
    for (i = 0; i < length; i++) {
        *dest = *dest OP data[i];
    }
}
```

- 消除不必要的内存引用
使用`acc`变量保存中间结果的值,现代处理器会优先考虑使用寄存器保存局部变量的值, 最后才把结果赋值给`*dest`,大大减少对内存的引用.可以成吨地提高效率。
```C
/* Accumulate result in local variable */
void combine4(vec_ptr v, data_t *dest)
{
    long int i;
    long int length = vec_length(v);
    data_t *data = get_vec_start(v);
    data_t acc = IDENT;

    for (i = 0; i < length; i++) {
        acc = acc OP data[i];
    }
    *dest = acc;
}
```

## 还能进一步提高性能吗?(CSAPP 3ed中文版357页)

上面优化手段都不依赖于目标机器的任何特性, 这些优化只是简单地降低了过程调用的开销，以及消除一些编译优化的妨碍因素。
所以我们平时写代码可以考虑较少内存访问,减少过程的调用以及编写编译器容易优化的友好代码。


如果试图进一步提高性能, 必须必须要理解现代处理器的微体系结构。