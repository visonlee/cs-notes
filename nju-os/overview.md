# 南京大学操作系统 蒋炎岩

- https://www.bilibili.com/video/BV1Cm4y1d7Ur
- https://jyywiki.cn/OS/2022/

## 什么是操作系统
我的理解是操作系统是对`图灵机`和`冯诺依曼结构`的实现, 造作系统本质上也是一个软件。它运行在最高的特权级(`ring 0`), 对下, 它负责管理硬件资源(内存、IO设备等),对上它负责管理用户程序(进程管理,提高系统调用API等)。

## 什么是程序

- 从硬件来看, CPU 上电后, 寄存器会reset成一个初始值(初始状态), 最重要的是PC($rip)寄存器, 然后系统不断从$rip取值, 然后根据该值去主存拿指令来执行, 然后根据计算结果来更新寄存器或者主存.
即`从PC取值 -> fetch 指令 -> execute 指令 -> update 寄存器或主存 -> 更新PC ->从PC取值 —> ...`
- 从C语言来理解程序状态机, C语言视觉来看, 其实就是变量和栈帧状态的变迁, 程序执行计算, 会变量的状态, 函数的调用会伴随栈帧状态的变迁(push or pop frame)

- 从二进制(汇编)来看状态机, 其状态 = 寄存器 + 内存。执行一条指令, 就会伴随状态的迁移。
  GDB调试器就是状态机的查看器,当程序停在某个断点时, gdb可以查看其当前的变量的状态, 当前的寄存器的值, 当前栈帧, 当前内存的值。

## 构建一个最小可执行的程序
```C
// a.c
void main() {}
```
`gcc a.c && ./a.out` 可以看到程序可以正常的执行和返回。
但是这里这里有几个问题.
```
> size a.out        
   text    data     bss     dec     hex filename
   1418     544       8    1970     7b2 a.out
> ls -l a.out 
-rwxr-xr-x 1 lws lws 16464 Sep 23 11:48 a.out
```
在这里我的C代码仅仅是一个空的`main`函数, 但是用`szie`命令可以看到`.text`,`.data`,`.bss`经常包含了这么多内容, 而`a.out`的可执行文件大小也将近17k.
说明`gcc`默认编译链接成`a.out`后, 里面除了`main`函数之外, 还包括了其他的`.text`(`objdump -j .text -D a.out`)和`.data`等,这个不符合我们最小可执行程序的预期。

`gcc`其实是一堆命令的集合, 其包括了`预处理`,`编译`,`链接`的过程, 我们可以使用`gcc --verbose`选项来查看编译链接的详细过程:
```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.4.0-1ubuntu1~20.04.1' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-9-Av3uEd/gcc-9-9.4.0/debian/tmp-nvptx/usr,hsa --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1) 
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/9/cc1 -quiet -v -imultiarch x86_64-linux-gnu a.c -quiet -dumpbase a.c -mtune=generic -march=x86-64 -auxbase a -version -fasynchronous-unwind-tables -fstack-protector-strong -Wformat -Wformat-security -fstack-clash-protection -fcf-protection -o /tmp/ccKJ3wyW.s
GNU C17 (Ubuntu 9.4.0-1ubuntu1~20.04.1) version 9.4.0 (x86_64-linux-gnu)
        compiled by GNU C version 9.4.0, GMP version 6.2.0, MPFR version 4.0.2, MPC version 1.1.0, isl version isl-0.22.1-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/9/include-fixed"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-linux-gnu/9/include
 /usr/local/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
GNU C17 (Ubuntu 9.4.0-1ubuntu1~20.04.1) version 9.4.0 (x86_64-linux-gnu)
        compiled by GNU C version 9.4.0, GMP version 6.2.0, MPFR version 4.0.2, MPC version 1.1.0, isl version isl-0.22.1-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
Compiler executable checksum: c0c95c0b4209efec1c1892d5ff24030b
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64'
 as -v --64 -o /tmp/cceXGF7S.o /tmp/ccKJ3wyW.s
GNU assembler version 2.34 (x86_64-linux-gnu) using BFD version (GNU Binutils for Ubuntu) 2.34
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/9/:/usr/lib/gcc/x86_64-linux-gnu/9/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/9/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/9/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/9/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/9/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/9/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper -plugin-opt=-fresolution=/tmp/cc4LQ8HW.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie -z now -z relro /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/Scrt1.o /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/9/crtbeginS.o -L/usr/lib/gcc/x86_64-linux-gnu/9 -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/9/../../.. /tmp/cceXGF7S.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-linux-gnu/9/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/crtn.o
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64'
```
从上面可以看到, 出来我们自己程序的本身, 在链接过程中, 我链接了程序所需要的系统库。

那我们能不能跳过上面编译链接的步骤,我们自己来构建了,下面来尝试一下。
```
> gcc -c a.c && objdump -j .text -D a.o #only complie a.c
a.o:     file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <main>:
   0:   f3 0f 1e fa             endbr64
   4:   55                      push   %rbp
   5:   48 89 e5                mov    %rsp,%rbp
   8:   90                      nop
   9:   5d                      pop    %rbp
   a:   c3                      retq
> ld a.o -o a.out
ld: warning: cannot find entry symbol _start; defaulting to 0000000000401000
```
这里找不到`symbol`,那我们hack一下,把源代码改成如下, `main` -> `_start`:
```C
// a.c
void _start() {}
```
然后执行如下命令
```
> gcc -c a.c && ld a.o
> size a.out
   text    data     bss     dec     hex filename
     99       0       0      99      63 a.out
> ls -lh a.out
-rwxr-xr-x 1 lws lws 9.0K Sep 23 14:51 a.out
> objdump -j .text -D a.out

a.out:     file format elf64-x86-64

Disassembly of section .text:
0000000000401000 <_start>:
  401000:       f3 0f 1e fa             endbr64
  401004:       55                      push   %rbp
  401005:       48 89 e5                mov    %rsp,%rbp
  401008:       90                      nop
  401009:       5d                      pop    %rbp
  40100a:       c3                      retq
> ./a.out
[1]    1210 segmentation fault  ./a.out
```

可以看到程序发生了segmentation fault.

那个程序错误的原因是什么呢, 我使用gdb调试之, `starti`让程序停留在第一条要执行的指令
```
➜  overview gdb -q ./a.out
Reading symbols from ./a.out...
(No debugging symbols found in ./a.out)
(gdb) starti
Starting program: /home/lws/os/njuos/overview/a.out

Program stopped.
0x0000000000401000 in _start ()
(gdb) layout asm

0x401000 <_start>       endbr64
0x401004 <_start+4>     push   %rbp
0x401005 <_start+5>     mov    %rsp,%rbp
0x401008 <_start+8>     nop
0x401009 <_start+9>     pop    %rbp
0x40100a <_start+10>    retq
...
────────────────────────────────────────────────────────────────
(gdb) si
0x0000000000401004 in _start ()
0x0000000000401005 in _start ()
0x0000000000401008 in _start ()
0x0000000000401009 in _start ()
0x000000000040100a in _start ()
(gdb) print $rsp
$1 = (void *) 0x7fffffffe0d0
(gdb) x $rsp
0x7fffffffe0d0: 0x00000001
(gdb) si
0x0000000000000001 in ?? ()
Cannot access memory at address 0x1
(gdb) si

Program received signal SIGSEGV, Segmentation fault.
0x0000000000000001 in ?? ()
```

当程序执行到`0x40100a <_start+10>    retq`, `retq`指令所执行的操作是, 帮`$rsp`栈顶的的值(这里是0x01)pop出, 赋值给`$rip`, 然后 `$rsp = $rsp + 8`。
这里明显`$rip = 0x1`存放着一个非法的指令地址了,所以程序就发生了`Segmentation fault`

那有什么办法让程序跑起来不报错了, 我们用while循环不让程序退出。
```C
// a.c
void _start() {while(1) {};}
```

```
gcc -c a.c && ld a.o 
```
可以看到程序可以跑起来了, 从这里我们也可以得出结论: C语言运行时库,帮我们setup好了我们进程所需要的环境。

那有什么办法可以在我们最小的可执行程序下, 退出程序呢, 答案是`syscall`.

```C
// gcc a.c && ./a.out
#include <sys/syscall.h>
int main() {
  syscall(SYS_exit, 42);
}
```
在这里, 调用了`syscall`, 在这个函数里面,最终调用了`syscall`指令执行系统调用.

或者我们使用汇编, 直接写`syscall`指令, 如下:
```s
#include <sys/syscall.h>

.globl _start
_start:
  movq $SYS_write, %rax   # write(
  movq $1,         %rdi   #   fd=1,
  movq $st,        %rsi   #   buf=st,
  movq $(ed - st), %rdx   #   count=ed-st
  syscall                 # );

  movq $SYS_exit,  %rax   # exit(
  movq $1,         %rdi   #   status=1
  syscall                 # );

st:
  .ascii "\033[01;31mHello, OS World\033[0m\n"
ed:
```

```
> gcc -c minimal.S && ld minimal.o && ./a.out    
Hello, OS World
```
可以看到, 程序正常打印,并且正常返回.

> 关于`syscall`的调用参数，调用号,  可以使用`man syscall`查看


## 什么是编译器
编译器是对源代码(状态机) -> 二进制代码(状态机)的翻译的程序, 并且它保证编译(优化)的正确性:

编译优化case:

- case1

```
> cat demo.c
void foo() {
    int a = 1; // 被优化了
}
>  gcc -c -O2 demo.c && objdump -d demo.o
demo.o:     file format elf64-x86-64

Disassembly of section .text:
0000000000000000 <foo>:
   0:   f3 0f 1e fa             endbr64
   4:   c3                      retq
```

- case2
```
> cat demo.c
extern int g;
void foo() {
    g++; // g是全局变量,不可以被删掉
    g++; // 但是可以把两次g++合并
}
> gcc -c -O2 demo.c && objdump -d demo.o
demo.o:     file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <foo>:
   0:   f3 0f 1e fa             endbr64
   4:   83 05 00 00 00 00 02    addl   $0x2,0x0(%rip)        # b <foo+0xb>
   b:   c3                      retq
```

- case3

```
> cat demo.c
extern int g;
void foo(int x) {
    g++; //只要在函数返回之前, 编译器依然可以优化合并两次g++为一次
    asm volatile ("nop" :: "r"(x)); // 这部分汇编不可以被优化
    g++;
}
> gcc -c -O2 demo.c && objdump -d demo.o
demo.o:     file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <foo>:
   0:   f3 0f 1e fa             endbr64
   4:   90                      nop
   5:   83 05 00 00 00 00 02    addl   $0x2,0x0(%rip)        # c <foo+0xc>
   c:   c3                      retq
```

- case4

```
➜  overview cat demo.c
extern int g;
void foo(int x) {
    g++;
    asm volatile ("nop" :: "r"(x) : "memory"); // complier barrier,两次g++不可以被合并
    // __sync_synchronize(); //也可以用这个, 对应与memory barrier的mfence指令
    g++;
}
➜  overview gcc -c -O2 demo.c && objdump -d demo.o

demo.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <foo>:
   0:   f3 0f 1e fa             endbr64
   4:   83 05 00 00 00 00 01    addl   $0x1,0x0(%rip)        # b <foo+0xb>
   b:   90                      nop
   c:   83 05 00 00 00 00 01    addl   $0x1,0x0(%rip)        # 13 <foo+0x13>
  13:   c3                      retq
```


## 操作系统眼中的一般程序

本质上, 所有的应用程序和helloworld没什么区别, 程序 = 计算 → syscall → 计算 → syscall → ...

可以使用`strace`跟踪程序的系统调用。
