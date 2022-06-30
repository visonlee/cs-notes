## gcc编译选项
- -Og 告诉编译器生成符合原始C代码整体结构的机器及代码


***

## Sizes of C data types in x86-64. With a 64-bit machine, pointers are 8 bytes long
| C declaration | Intel data type | Assembly-code suffifix | Size (bytes) |
| :----: | :----: | :----: | :----: |
|char    | | Byte           | b | 1 |
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