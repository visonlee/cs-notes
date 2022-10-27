## 操作系统的状态机模型

操作系统也是一个软件,也是一个C程序,也是状态机

我们之前写过一个`最小`的C程序
```S
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

这个程序不依赖任何的库,就可以在操作系统上运行起来了。

之前讲过`C程序`是一个状态机,这个没有问题

- 那么是谁把这个程序加载起来的呢? 当然是`操作系统`了
- 于是又有问题,那么谁去加载`操作系统`呢? 我们的程序可以直接在没有操作系统上的硬件吗?
- 操作系统是由`加载器`加载的,那么`加载器`又是谁去加载的呢

## 裸机(Bare-metal)与程序员的约定

为了让计算机能运行任何我们的程序，一定存在软件/硬件的约定

- CPU reset 后,处理器处于某个确定的状态
  - PC 指针一般指向一段`memory-mapped ROM`
  - ROM 存储了厂商提供的`firmware (固件)`
  - 处理器的大部分特性(如缓存、虚拟存储、……)处于关闭状态

- Firmware (固件，厂商提供的代码)
  - 将用户数据加载到内存
    - 例如存储介质上的第二级 loader (加载器)
    - 或者直接加载操作系统 (嵌入式系统)

## x86 Family: CPU Reset 行为
CPU Reset [Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3A/3B](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)

- 寄存器会有初始状态
  - EIP = 0x0000fff0
  - CR0 = 0x60000010
    - 16-bit 模式
  - EFLAGS = 0x00000002
    - interrupt disabled

![CPU Reset](./statc/intel-cpu-reset.png)

使用`qemu`验证`CPU`的`reset`状态,结果如下
```
> ~ #不启动图形界面, -S表示停在CPU reset状态 -s 表示开放1234端口给gdb调试
> ~ #退出gdb ctrl + a, x
> ~ qemu-system-i386 -nographic -S -s
> ~ gdb -q
(gdb) target remote localhost:1234
0x0000fff0 in ?? ()
(gdb) info registers
eax            0x0                 0
ecx            0x0                 0
edx            0x663               1635
ebx            0x0                 0
esp            0x0                 0x0
ebp            0x0                 0x0
esi            0x0                 0
edi            0x0                 0
eip            0xfff0              0xfff0
eflags         0x2                 [ IOPL=0 ]
cs             0xf000              61440
ss             0x0                 0
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x0                 0
gs_base        0x0                 0
k_gs_base      0x0                 0
cr0            0x60000010          [ CD NW ET ]
....... 省略
```

## CPU Reset之后：发生了什么？
- 从`PC (CS:IP)` 指针处`取指令`、`译码`、`执行`……
- 从`firmware`(`Legacy BIOS vs. UEFI`)开始执行
  - `ffff0` 通常是一条向`firmware`跳转的`jmp`指令, `jmp`固件(`BIOS`)的代码
  - 然后`BIOS`的固件代码检查硬件状态, 然后把可引导设备的前`512`字节(`0x55aa结尾`)的数据加载到内存`0x7c00`
    - 此时处理器处于`16-bit`模式
    - 规定 CS:IP = 0x7c00, (R[CS] << 4) | R[IP] == 0x7c00
      - 可能性1：CS = 0x07c0, IP = 0
      - 可能性2：CS = 0, IP = 0x7c00