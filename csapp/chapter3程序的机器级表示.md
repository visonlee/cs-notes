## gcc编译选项
- -Og 告诉编译器生成符合原始C代码整体结构的机器及代码


***

## x86-64 数据类型大小
Sizes of C data types in x86-64. With a 64-bit machine, pointers are 8 bytes long

| C declaration | Intel data type | Assembly-code suffifix | Size (bytes) |
| :----: | :----: | :----: | :----: |
|char    |   Byte           | b | 1 |
|short   | Word             | w | 2 |
|int     | Double word      | l | 4 |
|long    | Quad word        | q | 8 |
|char *  | Quad word        | q | 8 |
|float   | Single precision | s | 4 |
|double  | Double precision | l | 8 |

***
## Registers
|64-bit register | Lower 32 bits | Lower 16 bits | Lower 8 bits| |
| :----: | :----: | :----: | :----: | :----: |
|rax | eax | ax | al|return value|
|rbx | ebx | bx | bl|callee saved|
|rcx | ecx | cx | cl|4th argument|
|rdx | edx | dx | dl|3th argument|
|rsi | esi | si | sil|2th argument|
|rdi | edi | di | dil|1th argument|
|rbp | ebp | bp | bpl|callee saved|
|rsp | esp | sp | spl|stack pointer|
|r8 | r8d | r8w | r8b|5th argument|
|r9 | r9d | r9w | r9b|6th argument|
|r10 | r10d | r10w | r10b|caller saved|
|r11 | r11d | r11w | r11b|caller saved|
|r12 | r12d | r12w | r12b|callee saved|
|r13 | r13d | r13w | r13b|callee saved|
|r14 | r14d | r14w | r14b|callee saved|
|r15 | r15d | r15w | r15b|callee saved|

***
## 寻址方式
- `imm(ediate)` 立即数  `R(egister)`寄存器

| 类型 | 格式 | 操作数值 | 名称 |
| :----: | :----: | :----: | :----: |
| 立即数 | $Imm | Imm | 立即数寻址  |
| 寄存器 | ra | R[ra] | 寄存器寻址  |
| 存储器 | Imm | M[Imm] | 绝对(直接)寻址 |
| 存储器 | (ra) | M[R[ra]] | 间接寻址 |
| 存储器 | Imm(rb) | M[Imm + R[rb]] | (基址+偏移量)寻址 |
| 存储器 | (rb,ri) | M[R[rb] + R[ri]] | 变地寻址 |
| 存储器 | Imm(rb,ri) | M[Imm + R[rb] + R[ri]] | 变地寻址 |
| 存储器 | (,ri,s) | M[R[ri].s] | 比例变地寻址 |
| 存储器 | Imm(,ri,s) | M[Imm + R[ri].s] | 比例变地寻址 |
| 存储器 | (rb,ri,s) | M[R[rb] + R[ri].s] | 比例变地寻址 |
| 存储器 | Imm(rb,ri,s) | M[Imm + R[rb] + R[ri].s] | 比例变地寻址 |

- 例子
假设下main的值存放在指明的内存和寄存器中

| Address | Value | | Register | Value |
| :----: | :----: | :----: |  :----: |:----: |
| 0x100 | 0xFF | | %rax | 0x100 |
| 0x104 | 0xAB | | %rcx | 0x1 |
| 0x108 | 0x13 | | %rdx | 0x3 |
| 0x10C | 0x11 | |      |     |

下表是操作数所示的值

| 操作数 | 值 | 注释 |
| :----: | :----: | :----: |
| %rax           | 0x100  | 寄存器寻址 |
| 0x104          | 0xAB   | 直接(绝对)寻址 |
| $0x108         | $0x108 | 立即数寻址 |
| (%rax)         | 0xFF   | 间接寻址 |
| 4(%rax)        | 0xAB   |基址+偏移量寻址|
| 9(%rax,%rdx)   | 0x11   | 变址寻址 rax + rcx + 9 |
| 260(%rcx,%rdx) | 0x13   |变址寻址 rcx + rdx + 260 = 0x108|
| 0xFC(,%rcx,4)  | 0xFF   | 比例变地寻址, rcx * 4 + 0fc|
| (%rax,%rdx,4)  | 0x11   | 比例变地寻址, rax + rdx * 4 |



***
## push & pop 指令
- push %rbp 等价于: subq $8, $rsp; movq %rbp, (%rsp)
- popq %rax 等价于: movq $rax, %rax; addq $8, %rsp;

***
<br>
算术指令

|指令 | 结果 | 描述 |
| :----: | :----: | :----: |
| leaq S, D | D ← &S | Load effective address |
| inc D | D ← D+1 | Increment |
| dec D | D ← D−1 | Decrement |
| neg D | D ← -D | Negate |
| not D | D ← ~D | Complement |
| add S, D | D ← D + S | Add |
| sub S, D  | D ← D − S | Subtract |
| imul S, D | D ← D ∗ S | Multiply |
| xor S, D | D ← D ^ S | Exclusive-or |
| or S, D | D ← D or S | Or |
| and S, D | D ← D & S | And |
| sal k, D | D ← D << k | Left shift |
| shl k, D | D ← D << k | Left shift (same as sal) |
| sar k, D | D ← D >>A k | Arithmetic right shift |
| shr k, D | D ← D >>L k | Logical right shift |

## 加载有效地址
加载有效地址(load effective address)指令leaq实际上是movq指令的变形。这个指令看上去好像是从内存上读指定地址的数据到目标操作数中, 但实际上它并没有从内存读, 而是把该有效地址写入到目标操作数中。
用法:
- 可以为内存引用产生指针
- 还可以简洁地描述普通的运算操作。如: 假设%rdx的值为x, 那么leaq 7(%rdx, %rdx, 4), %rax. 则%rax等于 x + x * 4 + 7 = 5x+7

### leaq 与 movq 的区别
同样, 假设%rdx的值为x, 那么movq 7(%rdx, %rdx, 4), %rax. 则%rax等于从内存地址为 5x + 7 中的内存单元拿数据, 再赋值给%rax


***
## 条件码
- CF(Carry flag):进位标志。最近的操作使最高位产生了进位，可检查无符号数操作的溢出。
- ZF(Zero flag): 零标志,最近的操作结果得出零的结果
- SF(Signed flag): 符号标志,最近的操作结果为负数
- OF(Overflow flag0):一处标志,最近的操作导致一个补码正溢出或负溢出

比如, 一条add指令执行操作 t = a + b的功能(a、b、t睡是整型的)。然后结构根据下面的结果设置条件码:

| 标志 | 结果 | 注释 |
| :----: | :----: | :----: |
| CF | (unsigend) t < (unsigned)a  | 无符号溢出  |
| ZF | (t == 0 )                   | 零         | 
| SF | (t < 0)                     | 负数       | 
| OF | (a<0==b<0) && (t<0 !=a<0>)  | 有符号溢出  |

leaq 不会改变条件码, 其他指令INC,DECNEG,NOT, ADD,SUB,IMUL,XOR.OR,ANDSAL,SHL,SAR,SHR都会设置条件码。例如XOR, 进位和溢出标志都会被设置成0。对于移位操作, 进位标志经设置为最后一个被溢出的位,而溢出标志设置为0.INC和DEC会设置溢出和零标志, 但是不会改变进位标志位。

有些指令只设置条件码而不改变任何寄存器, 如CMP、TEST指令。
- CMP (CMP S1, S2,基于S2-S1)根据两个操作数只差来设置条件码,而不更新寄存器，如果两个操作数相等, Zero flag的结果为1。除此之外, 它是跟SUB指令是一样的。
- TEST TEST指令除了他们只设置条件码而不改变目其寄存器的值，其它与and指令一样。典型用法如(testq %rax, %rax来检查%rax是负数零,还是正数),或者其中一个操作数是掩码, 用来指示哪些位应该被测试

<br>

## 条件码访问
条件码访问有三种:
- 根据条件码组合, 将一个字节设置为1或者1 (SET指令)
- 条件跳转到程序的其他地方
- 有条件的传送数据

### set指令
一条set指令的目的操作数是低位单字节寄存器或一个字节的内存, 指令会将这个字节设置成0或1。为了得到一个32/64位的结果, 必须将高位清零。主义下面的setl的l是less, setb的b是below。
举个例子:
```asm
movq $0, %rax
movq $3, %rdi
movq $3, %rsi    # %rsi == %rdi
cmpq %rdi, %rsi  # 因为相等, 所以ZF为1,假如不相等ZF为0
setq %al         #把ZF=1赋值给%al,若ZF=0,也会把ZF=0赋值给%al
```
|指令 | 同义名 | 效果 | 条件 |
| :----: | :----: | :----: | :----: |
|sete D | setz | D ← ZF | 相等/零 |
|setne D | setnz | D ← ~ ZF | 不等/非零 |
|sets D | | D ← SF | 负数 | 
|setns D | | D ← ~ SF | 非负数 |
|setg D | setnle | D ← ~ (SF ^ OF) & ~ZF | 大于(有符号>)  |
|setge D | setnl | D ← ~ (SF ^ OF) | 大于或者等于 (有符号 >=) |
|setl D | setnge | D ← SF ^ OF | 小于 (有符号 <)
|setle D | setng | D ← (SF ^ OF) or ZF | 小于或等于 (有符号 <=) |
|seta D | setnbe | D ← ~ CF & ~ZF | 超过 (无符号 >) |
|setae D | setnb | D ← ~ CF | 超过或等于 (无符号 >=)  |
|setb D | setnae | D ← CF | 低于 (无符号 <)  |
|setbe D | setna | D ← CF or ZF | 低于或等于 (无符号 <=) |

### 跳转指令

| 指令 | 同义名 | 跳转条件 | 描述 |
| :----: | :----: | :----: | :----: |
| jmp Label | | 1 | unconditionally Direct jump |
| jmp *Operand | | 1 | unconditionally Indirect jump |
| je Label | jz | ZF | Equal / zero
| jne Label | jnz | ~ZF |  Not equal / not zero |
| js Label | | SF | Negative |
| jns Label | | ~SF | Nonnegative |
| jg Label | jnle | ~(SF ^ OF) & ~ZF | Greater (signed >) |
| jge Label | jnl | ~(SF ^ OF)  | Greater or equal (signed >=) |
| jl Label | jnge | SF ^ OF  | Less (signed <) |
| jle Label | jng | (SF ^ OF) or ZF  | Less or equal (signed <=) |
| ja Label | jnbe | ~CF & ~ZF  | Above (unsigned >) |
| jae Label | jnb | ~CF  | Above or equal (unsigned >=) |
| jb Label | jnae | CF | Below (unsigned <) |
| jbe Label | jna | CF or ZF | Below or equal (unsigned <=) |


- 相对寻址
假设有如下代码`branch.c`:
```c
long loop(long x)
{
    while (x > 0) {
        x = x >> 1;
    }
    return x;
}

int main()
{
    return loop(-2);
}
```
执行`gcc -Og -S branch.c` 得到`branch.s`, `loop`函数如下:
```asm
loop:
.LFB0:
        endbr64
        movq    %rdi, %rax
.L2:
        testq   %rax, %rax
        jle     .L4
        sarq    %rax
        jmp     .L2
.L4:
        ret
```
执行`gcc -C -Og branch.c`, 然后`objdump -d branch.o`得到
```asm
 0000000000000000 <loop>:
1   0:   f3 0f 1e fa             endbr64
2   4:   48 89 f8                mov    %rdi,%rax
3   7:   48 85 c0                test   %rax,%rax
4   a:   7e 05                   jle    11 <loop+0x11>
5   c:   48 d1 f8                sar    %rax
6   f:   eb f6                   jmp    7 <loop+0x7>
7  11:   c3                      retq
```
第4行中跳转指令的目标指明为`0x11`,第6行中跳转指令的目标指明为`0x07`.
不过观察指令的编码,会看到第一条指令的目标编码(在第二个字节中)为`0x05`, 把它加上`0x0c`(因为目标编码`0x05`距离函数`loop`开头为 `0x0c`)，即`0x11 = 0x05 + 0x0c`,即第7行处的指令
类似地, 第二个跳转指令的目标用单字节补码表示为`0xF6`(十进制-10)。将这个数数加上`0x11`, 即 `0x07 = 0x11 - 0x0a`,即跳转第3行的指令
下面是链接后反汇编的代码
```asm
 0000000000001129 <loop>:
1    1129:       f3 0f 1e fa             endbr64
2    112d:       48 89 f8                mov    %rdi,%rax
3    1130:       48 85 c0                test   %rax,%rax
4    1133:       7e 05                   jle    113a <loop+0x11>
5    1135:       48 d1 f8                sar    %rax
6    1138:       eb f6                   jmp    1130 <loop+0x7>
7    113a:       c3                      retq
```
这些指令被重定位到不同的地址, 但是第4和第6行的跳转目标编码并没有变。通过使用与PC相对的跳转目标编码, 指令只需要很简洁(只需要2个自己), 而且目标代码可以不做改变就能移到内存中不同的位置。


- 绝对寻址
 todo

<br>
<br>

 ### 有条件的数据传送
来看两个例子:
- 条件控制实现条件分支
来看一段代码
```C
long lt_cnt = 0;
long ge_cnt = 0;

long absdiff_se(long x, long y)
{
    long result;
    if (x < y) {
      lt_cnt++;
      result = y - x;
    } else {
     ge_cnt++;
     result = x - y;
    }
    return result;
}
```
其对应的汇编代码控制流程类似下面这段C代码
```C
long gotodiff_se(long x, long y)
{
    long result;
    if (x >= y)
        goto x_ge_y;
    lt_cnt++;
    result = y - x;
    return result;
x_ge_y:
    ge_cnt++;
    result = x - y;
    return result;
}
```
产生的汇编代码 `-O2` 编译选项
```asm
absdiff_se:
.LFB0:
        endbr64
        movq    %rsi, %rax
        cmpq    %rsi, %rdi
        jge     .L2
        addq    $1, lt_cnt(%rip)
        subq    %rdi, %rax
        ret
.L2:
        subq    %rsi, %rdi
        addq    $1, ge_cnt(%rip)
        movq    %rdi, %rax
        ret
```
在这里因为`lt_cnt`和`ge_cnt`这两个全局变量存在而产生副作用, 编译器没有办法做进一步的优化。这种条件控制的方法,当条件满足是,程序沿着一条路径执行, 当条件不满足时, 就走另外一条路径。这种机制简单而通用, 但是在现代处理器上, 它可能会非常低效。

<br>

- 条件传送实现条件分支
再看一个例子:
```C
long absdiff(long x, long y)
{
    long result;
    if (x < y)
      result = y - x;
    else
      result = x - y;
    return result;
}
```
其实现逻辑对应的C代码:
```C
long cmovdiff(long x, long y)
{
    long rval = y - x;
    long eval = x - y;
    long ntest = x >= y;
    if (ntest) rval = eval; //这一行需要单指令实现, 即cmov指令
    return rval;
}
```
产生的汇编代码 `-O2` 编译选项
```asm
# long absdiff(long x, long y)
# x in %rdi, y in %rsi
absdiff:
        endbr64
        movq %rsi, %rax
        subq %rdi, %rax  #rval = y-x
        movq %rdi, %rdx
        subq %rsi, %rdx  #eval = x-y
        cmpq %rsi, %rdi  #Compare x:y
        cmovge %rdx, %rax #If >=, rval = eval
        ret Return tval
```
可以看到,因为这个例子中不像条件控制实现条件分支的那个例子, 那个例子中因为`lt_cnt`和`ge_cnt`这两个全局变量而无法优化。而当前这里例子, 因为没有全局变量, 所以把`if`和`else`都算一遍都没有问题, 然后使用`cmovge`实现条件传送.

以下内容摘自CSAPP3e中文译本 Page 146
> 为了理解为什么基于条件数据传送的代码会比基于条件控制转移的代码性能要好，我们必须了解一些关于现代处理器如何运行的知识。正如我们在第4章和第5章中看到的，处理器通过流水线（pipelining）来获得高性能，在流水线中，一条指令的处理要经过一些列的阶段，每个阶段执行所需操作的一小部分（例如，从内存取指令，确定指令类型，从内存读数据，执行算术运算，向内存写数据，以及更新程序计数器）。这种方法通过重叠连续指令的步骤来获得高性能，例如，在取一条命令的同时，执行它前面一条指令的算术运算。要做到这一点，要求能够事先确定要执行的指令序列，这样才能保持流水线中充满了待执行的指令。当机器要到条件跳转（也称为“分支”）时，只有当分支条件求值完成后，才能决定分支往哪边走，处理器采用非常精密的分支预测逻辑来猜测每条跳转指令是否会执行。只要它的猜测还比较可靠（现代微处理器设计试图达到90%以上的成功率），指令流水线就会充满着指令。另一方面，错误预测一个跳转，要求处理器丢掉它为该跳转指令后所有指令已做的工作，然后再开始从正确位置其实的指令取填充流水线，正如我们会看到的，这样一个错误预测会招致很严重的处罚，浪费大约15~30个时钟周期，导致程序性能严重下降。

<br>

| 指令 | 同义名 | 传送条件 |  描述 |
| :----: | :----: | :----: | :----: |
| cmove S, R | cmovz | ZF | Equal / zero |
| cmovne S, R | cmovnz | ~ZF | Not equal / not zero |
| cmovs S, R | |SF | Negative |
| cmovns S, R | | ~SF | Nonnegative |
| cmovg S, R | cmovnle | ~(SF ^ OF) & ~ZF|  Greater (signed >) |
| cmovge S, R | cmovnl | ~(SF ^ OF) | Greater or equal (signed >=) |
| cmovl S, R | cmovnge | SF ^ OF | Less (signed <) |
| cmovle S, R | cmovng | (SF ^ OF) or ZF | Less or equal (signed <=) |
| cmova S, R | cmovnbe | ~CF & ~ZF |  Above (unsigned >) |
| cmovae S, R | cmovnb | ~CF | Above or equal (Unsigned >=) |
| cmovb S, R | cmovnae | CF | Below (unsigned <) |
| cmovbe S, R | cmovna | CF | ZF Below or equal (unsigned <=) |


当然条件传送也不是完美的， 

- 比如下面代码:
```C
long cread(long *xp) {
    return (xp ? *xp : 0);
}
```
因为存在空指针引用的问题, 就只能使用条件控制来编译这段代码.

- 再比如, `if` 和 `else` 都需要大量的计算, 如果条件不满足, 那么这些计算也就白白浪费了.

***
<br>

## 循环
`do-while`,`while`,`for`循环，略


## switch
编译器对switch的优化, 看以下代码:

```C
void switch_eg(long x, long n, long *dest)
{
    long val = x;
    switch (n)
    {
    case 100:
        val *= 13;
        break;
    case 102:
        val += 10;
    /* Fall through */
    case 103:
        val += 11;
        break;
    case 104:
    case 106:
        val *= val;
        break;
    default:
        val = 0;
    }
    *dest = val;
}
```

翻译到拓展的C语言:
```C
void switch_eg_impl(long x, long n, long *dest)
{
    /* Table of code pointers */
    static void *jt[7] = {
        &&loc_A, &&loc_def, &&loc_B,
        &&loc_C, &&loc_D, &&loc_def,
        &&loc_D};
    unsigned long index = n - 100;
    long val;

    if (index > 6)
        goto loc_def;
    /* Multiway branch */
    goto *jt[index];

loc_A: /* Case 100 */
    val = x * 13;
    goto done;
loc_B: /* Case 102 */
    x = x + 10;
/* Fall through */
loc_C: /* Case 103 */
    val = x + 11;
    goto done;
loc_D: /* Cases 104, 106 */
    val = x * x;
    goto done;
loc_def: /* Default case */
    val = 0;
done:
    *dest = val;
}
```

其对应的汇编代码
```asm
# void switch_eg(long x, long n, long *dest)
# x in %rdi, n in %rsi, dest in %rdx
switch_eg:
    subq $100, %rsi Compute index = n-100
    cmpq $6, %rsi Compare index:6
    ja .L8 If >, goto loc_def
    jmp *.L4(,%rsi,8) Goto *jg[index]
  .L3: loc_A:
    leaq (%rdi,%rdi,2), %rax 3*x
    leaq (%rdi,%rax,4), %rdi val = 13*x
    jmp .L2 Goto done
  .L5: loc_B:
    addq $10, %rdi x = x + 10
  .L6: loc_C:
    addq $11, %rdi val = x + 11
    jmp .L2 Goto done
  .L7: loc_D:
    imulq %rdi, %rdi val = x * x
    jmp .L2 Goto done
  .L8: loc_def:
    movl $0, %edi val = 0
  .L2: done:
    movq %rdi, (%rdx) *dest = val
    ret Return
```
可以看到,在这个例子中, 编译器用一个跳转表来优化switch语句

---
<br>

# 过程

### 运行时栈

![stack](./static/stackframe.PNG)

以下内容摘自csapp第三版中文版第164页并作适当删减

>C语言过程调用机制的一个关键特性(大多数语言也是如此)在于使用了栈数据结构提供的后进先出的
>内存管理原则. 在过程P调用过程Q的例子中, 当Q在执行时, P以及所有向上追溯到P的调用链的过程,
>都时暂时被挂起的.当Q运行时, 只需要为局部变量分配新的存储空间, 或者设置到另一个过程的调用.另一方面, 当Q返回时,任何它所分配的局部存储空间都可以被释放. 因此程序可以用栈来管理它的过程所需要的存储空间, 栈和程序寄存器存放着传递控制和数据、分配内存所需要的信息.当P调用Q时，控制和数据信息被添加到堆栈的末尾。当P返回时，此信息将被释放。

>x86-64的栈向低地址增长, 栈顶指针rsp指向栈顶元素. 将栈指针减小一个适当的量可以为没有指定初始值的数据在栈上分配空间(我们会在汇编代码中经常看到). 类似的, 可以通过增加栈指针来释放空间.

> 当x86-64过程需要的存储空间超过寄存器能够存放的大小时, 会在栈上分配空间, 这部分称为过程的栈帧.上图给出了运行时栈的通用结构，包括把它划分为栈帧, 可以看到正在执行的过程(Q)总是在栈顶, 其上按调用顺序包括过程P和之前的过程. 当过程P调用过程Q时, 会把返回地址压入栈中, 指明了当Q返回时, 要从P程序的哪个位置继续执行. 我们把这个返回地址作为过程P的栈帧的一部分, 因为它存放的是和P相关的状态.Q的代码会扩展当前栈的边界, 分配它的栈帧所需的空间. 在这个空间中, 它可以保存寄存器的值, 分配局部变量空间, 为它调用的过程设置参数. 大多数过程的栈帧都是定长的, 在过程开始时就分配好了.但有些过程需要边长的帧, 这个问题会在3.10.5讨论.过程P最多可以在堆栈上传递六个整数值（即指针和整数），但如果Q需要更多参数，则可以在调用之前由 P 将这些参数存储在其堆栈帧中。

> 为了提高空间和时间效率, x86-64过程只分配自己所需要的栈帧部分. 如果过程需要6个或更少的参数, 那么
所有参数可以通过寄存器传递; 此外如果过程不需要保存返回地址(不调用其他过程, 有时称为叶子过程),
这样的过程就不需要栈帧.



### CALL 和RET指令
还是用一段代码和GDB调试说明吧


```C
// stackframe.c
long foo(long a) {
   long c = a + a;
   return c;
}

int main() {
    long x = 0xaa;
    long ret = foo(x);
    return 0;
}
```
<br>

- 64位的情况

使用`gcc -O0 stackframe.c  && objdump -d ./a.out` 后部分的反汇编如下:

```asm
0000000000001129 <foo>:
    1129:       f3 0f 1e fa             endbr64
    112d:       55                      push   %rbp                #保存rbp的副本
    112e:       48 89 e5                mov    %rsp,%rbp           #把rsp的值复制给rbp,用来保存当前栈帧的基地址
    1131:       48 89 7d e8             mov    %rdi,-0x18(%rbp)    #foo函数参数rdi赋值给-0x18(%rbp),即参数a
    1135:       48 8b 45 e8             mov    -0x18(%rbp),%rax    # 把参数a赋值给 rax
    1139:       48 01 c0                add    %rax,%rax           # rax = rax * 2
    113c:       48 89 45 f8             mov    %rax,-0x8(%rbp)     # 把rax赋值给-0x8(%rbp), 即变量c
    1140:       48 8b 45 f8             mov    -0x8(%rbp),%rax     # 把变量c赋值给rax,rax保存的是foo函数的返回值
    1144:       5d                      pop    %rbp                #恢复现场
    1145:       c3                      retq                       # 相当于:pop栈顶数据(即上个函数的返回地址)并赋值给rip

0000000000001146 <main>:
    1146:       f3 0f 1e fa             endbr64
    114a:       55                      push   %rbp                 #保存rbp的副本
    114b:       48 89 e5                mov    %rsp,%rbp            #把rsp的值复制给rbp,用来保存当前栈帧的基地址
    114e:       48 83 ec 10             sub    $0x10,%rsp           #分配0x10(十进制16)字节栈空间,栈往下生长 
    1152:       48 c7 45 f0 aa 00 00    movq   $0xaa,-0x10(%rbp)    # 把0xaa赋值给-0x10(%rbp),即变量x
    1159:       00
    115a:       48 8b 45 f0             mov    -0x10(%rbp),%rax     #把变量x的值赋值给rax
    115e:       48 89 c7                mov    %rax,%rdi            #再赋值给rdi, rdi保存的是foo函数的第一个参数
    1161:       e8 c3 ff ff ff          callq  1129 <foo>           #call相当于1:当前的指令的下一条压栈 2:foo函数地址赋值给rip
    1166:       48 89 45 f8             mov    %rax,-0x8(%rbp)      #rax是foo函数的返回值, 赋值给-0x8(%rbp)，即变量ret
    116a:       b8 00 00 00 00          mov    $0x0,%eax            #main函数的返回参数0赋值给eax
    116f:       c9                      leaveq                      #恢复现场,leaveq等价于这两条:mov %rbp, %rsp; pop %rbp
    1170:       c3                      retq                        #返回(return)指令
```
我把gdb初始化的命令放在`gdbinit`文件下, 如下:
```
b main
layout asm
layout regs
r
```
然后执行`gdb ./a.out -q -x ./gdbinit`
```
(gdb) p $rsp
$4 = (void *) 0x7fffffffe118
(gdb) x/xg $rsp
0x7fffffffe118: 0x00007ffff7de7083
(gdb) x/-10xg (0x7fffffffe118 + 8)
0x7fffffffe0d0: 0x00007fffffffe0f6      
0x7fffffffe0d8: 0x00005555555551cd
0x7fffffffe0e0: 0x00007ffff7fb42e8
0x7fffffffe0e8: 0x0000555555555180
0x7fffffffe0f0: 0x0000000000000000
0x7fffffffe0f8: 0x0000555555555040
0x7fffffffe100: 0x00007fffffffe200   
0x7fffffffe108: 0x0000000000000000
0x7fffffffe110: 0x0000555555555180
0x7fffffffe118: 0x00007ffff7de7083
(gdb)
```
上面GDB调试结果是我进入`main`函数的`栈帧`情况,然后执行 `si` (single instruction)执行单步调试.
为了方便,我画栈帧地址从高到低排列(用`sort --reverse`),下面是main函数返回前栈内存的情况:

| 地址 | 值 | 描述 |
| :----: | :----: | :----: |
| 0x7fffffffe118 | 0x00007ffff7de7083 | main函数进来时rsp的初始值|
| 0x7fffffffe110 | 0x0000000000000000 | 这里保存的是rbp的备份,main函数栈帧的开始|
| 0x7fffffffe108 | 0x154              | main函数变量ret|
| 0x7fffffffe100 | 0xaa               | main函数的变量x |
| 0x7fffffffe0f8 | 0x555555555166     | call <foo>的下一条指令的地址|
| 0x7fffffffe0f0 | 0x7fffffffe110     | 备份main函数的栈帧的基地址,foo函数栈帧的开始 |
| 0x7fffffffe0e8 | 0x154              | foo函数返回值,即foo函数c变量的值| 
| 0x7fffffffe0e0 | 未使用              | 未使用 |
| 0x7fffffffe0d8 | 0xaa               | foo函数的参数a |
| 0x7fffffffe0d0 | 未使用              | 未使用 |

<br>

- 32位的情况

使用`gcc -m32 -O0 stackframe.c  && objdump -d ./a.out` 后部分的反汇编如下:
```asm
000011ad <foo>:
    11ad:       f3 0f 1e fb             endbr32
    11b1:       55                      push   %ebp                           #保存ebp的副本,即保存main函数栈帧的基地址
    11b2:       89 e5                   mov    %esp,%ebp                      #把esp的值复制给ebp,用来保存当前栈帧的基地址
    11b4:       83 ec 10                sub    $0x10,%esp                     #分配0x10(十进制16)字节栈空间,栈往下生长
    11b7:       e8 42 00 00 00          call   11fe <__x86.get_pc_thunk.ax>   #忽略,不在讨论范围
    11bc:       05 20 2e 00 00          add    $0x2e20,%eax                   #eax = eax + $0x2e20
    11c1:       8b 45 08                mov    0x8(%ebp),%eax                 #相当于foo函数参数a赋值给eax
    11c4:       01 c0                   add    %eax,%eax                      #eax = eax * 2
    11c6:       89 45 fc                mov    %eax,-0x4(%ebp)                #-0x4(%ebp)是变量c
    11c9:       8b 45 fc                mov    -0x4(%ebp),%eax                #返回值传给eax
    11cc:       c9                      leave
    11cd:       c3                      ret

000011ce <main>:
    11ce:       f3 0f 1e fb             endbr32
    11d2:       55                      push   %ebp                           #保存ebp的副本
    11d3:       89 e5                   mov    %esp,%ebp                      #把esp的值复制给ebp,用来保存当前栈帧的基地址
    11d5:       83 ec 10                sub    $0x10,%esp                     #分配0x10(十进制16)字节栈空间,栈往下生长
    11d8:       e8 21 00 00 00          call   11fe <__x86.get_pc_thunk.ax>   #忽略,不在讨论范围
    11dd:       05 ff 2d 00 00          add    $0x2dff,%eax                   #eax = eax + $0x2dff
    11e2:       c7 45 f8 aa 00 00 00    movl   $0xaa,-0x8(%ebp)               # 变量x = 0xaa
    11e9:       ff 75 f8                pushl  -0x8(%ebp)                     # 把参数a压栈,传给foo函数
    11ec:       e8 bc ff ff ff          call   11ad <foo>                     #call指令的下一条指令压栈,eip指向foo函数
    11f1:       83 c4 04                add    $0x4,%esp                      #函数传参用栈,这里相当于对传参栈的回收
    11f4:       89 45 fc                mov    %eax,-0x4(%ebp)                #eax接收foo函数的返回值, 并传给变量ret
    11f7:       b8 00 00 00 00          mov    $0x0,%eax                      #main函数的返回参数0赋值给eax
    11fc:       c9                      leave                                 #恢复现场
    11fd:       c3                      ret                                   #返回(return)指令
```

也一样, 然后执行`gdb ./a.out -q -x ./gdbinit`

```
(gdb) p $esp
$1 = (void *) 0xffffd27c
(gdb) x/xw $esp
0xffffd27c:     0xf7de2ed5
(gdb) x/-10xw (0xffffd27c + 4)
0xffffd258: 0xffffd31c   
0xffffd25c: 0x56556231    
0xffffd260: 0xf7fe22d0   
0xffffd264: 0x00000000
0xffffd268: 0x00000000    
0xffffd26c: 0x00000000     
0xffffd270: 0xf7fb0000    
0xffffd274: 0xf7fb0000
0xffffd278: 0x00000000      
0xffffd27c: 0xf7de2ed5
(gdb
```
同样, 上面GDB调试结果是我进入`main`函数的`栈帧`情况,然后执行 `si` (single instruction)执行单步调试.
为了方便,我画栈帧地址从高到低排列(用`sort --reverse`),下面是main函数返回前栈内存的情况:

| 地址 | 值 | 描述 |
| :----: | :----: | :----: |
| 0xffffd27c | 0xf7de2ed5 | main函数进来时esp的初始值 |
| 0xffffd278 | 0x00000000 | 这里保存的是ebp的备份,main函数栈帧的开始 |
| 0xffffd274 | 0x154      | main函数变量ret |
| 0xffffd270 | 0xaa       | main函数的变量x |
| 0xffffd26c | 0x00000000 | 未使用 |
| 0xffffd268 | 0x00000000 | 未使用 |
| 0xffffd264 | 0xaa       | 变量x压栈, 作foo函数参数 |
| 0xffffd260 | 0x565561f1 | call foo指令的下一条指令地址 |
| 0xffffd25c | 0xffffd278 | 备份main函数的栈帧的基地址,foo函数栈帧的开始 |
| 0xffffd258 | 0x154      | foo函数变量c |

> 总结:可以看到通用的代码, x64使用寄存器传参数(当函数个数小于等于6), 生成的汇编代码更加容易读懂, 并且使用寄存器传参数,效率会比基于栈传递效率高



***
<br>

## 内存越界引用和缓冲区溢出

看如下例子:
```C
#include <stdio.h>
/* strcpy的自定义实现 */
void mystrcpy(char *dest, const char *src)
{
    if(dest == NULL || src == NULL) return; 

    while ((*dest++ = *src++) != '\0'); 
}

int main(int argc, char const *argv[])
{
    char *src = "12345";
    char dest[4]; //缓冲区太小

    mystrcpy(dest, src);
    puts(dest);
    
    return 0;
}
```
`gcc demo.c && ./a.out`,然后报下面的错误:
```
12345
*** stack smashing detected ***: terminated
[1]    3919 abort      ./a.out
```

下面是`main`函数反汇编结果分析:(`objdump -d ./a.out`)
```asm
00000000000011b1 <main>:
    11b1:       f3 0f 1e fa             endbr64
    11b5:       55                      push   %rbp
    11b6:       48 89 e5                mov    %rsp,%rbp
    11b9:       48 83 ec 30             sub    $0x30,%rsp          # 分配栈空间
    11bd:       89 7d dc                mov    %edi,-0x24(%rbp)    
    11c0:       48 89 75 d0             mov    %rsi,-0x30(%rbp)
    11c4:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax       # 段寻址,金丝雀值
    11cb:       00 00
    11cd:       48 89 45 f8             mov    %rax,-0x8(%rbp)     # 保存金丝雀值到栈中, 用作栈保护
    11d1:       31 c0                   xor    %eax,%eax           # 异或, 清零
    11d3:       48 8d 05 2a 0e 00 00    lea    0xe2a(%rip),%rax    # 2004 <_IO_stdin_used+0x4> 字符串"12345"首地址给rax
    11da:       48 89 45 e8             mov    %rax,-0x18(%rbp)    # 变量src
    11de:       48 8b 55 e8             mov    -0x18(%rbp),%rdx    # rdx存的是src变量
    11e2:       48 8d 45 f4             lea    -0xc(%rbp),%rax     # -0xc(%rbp) dest缓冲区
    11e6:       48 89 d6                mov    %rdx,%rsi           # 第二个参数 src
    11e9:       48 89 c7                mov    %rax,%rdi           # 第一个参数 dest
    11ec:       e8 78 ff ff ff          callq  1169 <mystrcpy>     # 调用 call 函数
    11f1:       48 8d 45 f4             lea    -0xc(%rbp),%rax     # call puts函数准备
    11f5:       48 89 c7                mov    %rax,%rdi           # call puts函数准备
    11f8:       e8 63 fe ff ff          callq  1060 <puts@plt>     # call puts
    11fd:       b8 00 00 00 00          mov    $0x0,%eax
    1202:       48 8b 4d f8             mov    -0x8(%rbp),%rcx     # 把之前存的金丝雀值放到rxc
    1206:       64 48 33 0c 25 28 00    xor    %fs:0x28,%rcx       # 两个金丝雀值比较一下
    120d:       00 00
    120f:       74 05                   je     1216 <main+0x65>    # 如果相等,说明-0x8(%rbp)没有被破坏, 即缓冲区没有溢出，那么正常返回
    1211:       e8 5a fe ff ff          callq  1070 <__stack_chk_fail@plt> # 如果来到这里,说明金丝雀-0x8(%rbp)被破坏,异常退出
    1216:       c9                      leaveq
    1217:       c3                      retq
    1218:       0f 1f 84 00 00 00 00    nopl   0x0(%rax,%rax,1)
    121f:       00
```

上面可以看到，通过分析mian函数栈帧,如下图:

| 地址 | 值 | 描述 |
| :----: | :----: | :----: |
| ... | ...       |  |
| -0x8(%rbp) | 金丝雀值 | 来自%fs:0x28,用于栈保护 |
|  -0xc(%rbp)| [3][2][1][0] | dest缓冲区 |
| ... | ...      |  |

可以看到, 因为`dest`缓冲区的大小只有4个字节, 而`src`字符串`12345`放不下, 最终会导致`-0x8(%rbp)`上的金丝雀地址被覆盖了。最终`xor %fs:0x28,%rcx`比较结果不一样而调用`__stack_chk_fail`函数。
同时这里也告诉我们`strcpy`函数不安全, 用得不好会导致程序问题, 可以用`strncpy`代替

我们可以用编译选项`-fno-stack-protector`,来看下没有栈保护的情况,`gcc -fno-stack-protector demo.c`,然后再看汇编(反汇编略)看`main`也没有栈保护和调用`__stack_chk_fail`的相关代码，在这里,程序也确实可以正常打印。但是在实际工作中不要这样做,因为这是一个UB(undefined behavior),我们加上栈保护功能,能是我们的代码更加安全,同时这里看到也只有非常小的性能损失, 所以加上栈保护行为是非常有必要的。

除了栈保护检测来对抗缓冲区溢出攻击, 其他防护手段还有:
- 栈随机化
如:
```C
void foo() {
    long local;
    printf("%p\n", &local); //程序每次运行,打印的结果都不一样,可以有效保护程序
}
```
- 限制可执行代码区域
略(请参考csapp第三步中文版201页)