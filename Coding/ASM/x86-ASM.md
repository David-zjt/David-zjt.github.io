## 一、x86 和 x86-64 汇编语言概述

x86 是一种 32 位的 CPU 指令集架构，最初由 Intel 在 1985 年推出，用于 80386 处理器。x86-64（或称为 x64）是其扩展，支持 64 位的计算，通常用于现代的 Intel 和 AMD 处理器（如 Intel Core i7 / i9，AMD Ryzen 等）。x86 和 x86-64 汇编语言是针对这些架构的低级编程语言。

---

## 二、基础语法

x86 汇编语言通常使用**AT&T 汇编语法**或**Intel 汇编语法**。不同的汇编器（如 NASM、GAS、MASM）支持不同的语法形式。

### 1. Intel 汇编语法（常用在 MASM 中）

- **操作数顺序**：源操作数在前，目标操作数在后。
- **寄存器写法**：使用大写字母，如 `EAX`、`EBX`、`ESP`、`RAX`、`RBX`、`RSP` 等。
- **寻址方式**：如 `mov eax, [ebx + 4]` 表示将 `ebx + 4` 的内存地址的内容移动到 `eax`。
- **指令格式**：`opcode dst, src`

---

### 2. AT&T 汇编语法（常用在 GAS 中）

- **操作数顺序**：目标操作数在前，源操作数在后。
- **寄存器写法**：使用小写字母，如 `eax`、`ebx`、`rsp`。
- **寻址方式**：使用 `$` 表示立即数，`%` 表示寄存器，如 `movl $42, %eax`。
- **指令格式**：`opcode src, dst`

---

## 三、寄存器架构

### 1. x86（32 位）寄存器

x86 架构包含 8 个通用寄存器：

| 寄存器 | 位数 | 用途 |
|--------|------|------|
| EAX    | 32   | 通用寄存器，常用于累加器 |
| EBX    | 32   | 通用寄存器，常用于基址寄存器 |
| ECX    | 32   | 通用寄存器，常用于计数器 |
| EDX    | 32   | 通用寄存器，常用于数据寄存器 |
| ESI    | 32   | 源索引寄存器 |
| EDI    | 32   | 目标索引寄存器 |
| ESP    | 32   | 堆栈指针（Stack Pointer） |
| EBP    | 32   | 基址指针（Base Pointer） |

此外还有 16 位寄存器（如 AX、BX、CX、DX）作为 EAX 等的别名，也可以使用。

### 2. x86-64（64 位）寄存器

x86-64 增加了 8 个新的通用寄存器（64 位），并扩展了原有的寄存器为 64 位（如 RAX、RBX 等）。

| 寄存器 | 位数 | 用途 |
|--------|------|------|
| RAX    | 64   | 通用寄存器，常用于累加器 |
| RBX    | 64   | 通用寄存器，基址寄存器 |
| RCX    | 64   | 通用寄存器，计数器 |
| RDX    | 64   | 通用寄存器，数据寄存器 |
| RSI    | 64   | 源索引寄存器 |
| RDI    | 64   | 目标索引寄存器 |
| RSP    | 64   | 堆栈指针 |
| RBP    | 64   | 基址指针 |

此外还有 16 位和 32 位的别名（如 AX、EAX、RAX）。

#### 特殊寄存器（Segment Registers）

| 寄存器 | 位数 | 用途 |
|--------|------|------|
| CS     | 16   | 代码段寄存器 |
| DS     | 16   | 数据段寄存器 |
| SS     | 16   | 堆栈段寄存器 |
| ES     | 16   | 扩展段寄存器 |
| FS     | 16   | 系统段寄存器 |
| GS     | 16   | 其他系统段寄存器 |

这些寄存器在保护模式和虚拟模式中使用较多。

#### 控制寄存器（Control Registers）

| 寄存器 | 位数 | 用途 |
|--------|------|------|
| CR0    | 32   | 控制处理器的全局状态（如保护模式启用） |
| CR2    | 32   | 指向当前正在执行的页表项 |
| CR3    | 32   | 指向页目录表 |
| CR4    | 32   | 控制高级 CPU 功能（如分页、SSE 等） |

---

## 四、主要汇编指令分类

x86 和 x86-64 架构的指令可以分为几大类：

### 1. 数据传输指令（Move）

- `mov`：移动数据
  - `mov eax, ebx` 将 `ebx` 的内容复制到 `eax`
  - `mov [eax], ebx` 将 `ebx` 的内容复制到 `eax` 指向的内存地址
- `push`：将数据压入堆栈
- `pop`：将数据弹出堆栈
- `lea`：加载有效地址（Load Effective Address）

### 2. 算术运算指令（Arithmetic）

- `add`：加法
- `sub`：减法
- `mul`：无符号乘法
- `imul`：带符号乘法
- `div`：无符号除法
- `idiv`：带符号除法
- `inc`：加 1
- `dec`：减 1
- `neg`：取反
- `and`：按位与
- `or`：按位或
- `xor`：按位异或
- `not`：按位取反

### 3. 逻辑运算指令（Logic）

- `and`, `or`, `xor`, `not`（见上）
- `test`：逻辑与并测试标志位
- `cmp`：比较两个操作数（不改变操作数，只影响标志位）

### 4. 移位操作指令（Shift）

- `shl` / `sal`：逻辑左移 / 算术左移（等价）
- `shr`：逻辑右移
- `sar`：算术右移（符号位扩展）

### 5. 控制流指令（Control Flow）

- `jmp`：无条件跳转
- `call`：调用子程序
- `ret`：从子程序返回
- `je` / `jne`：条件跳转（相等/不相等）
- `jz` / `jnz`：条件跳转（零/非零）
- `jl` / `jle` / `jg` / `jge`：条件跳转（小于、小于等于、大于、大于等于）
- `loop`：循环（根据 `ecx` 的值）

### 6. 系统操作指令（System）

- `int`：软中断
- `iret`：中断返回
- `hlt`：挂起处理器（停止执行）

---

## 五、常用汇编指令示例（Intel 语法）

### 1. 数据传输

```asm
mov eax, 5        ; 将 5 赋值给 EAX
mov ebx, [eax]    ; 将 EAX 指向的内存内容复制到 EBX
push eax          ; 将 EAX 压入堆栈
pop ecx           ; 将堆栈内容弹出到 ECX
lea edx, [ebx + 4] ; 将 ebx+4 的地址加载到 EDX
```

### 2. 算术运算

```asm
add eax, ebx      ; EAX = EAX + EBX
sub ebx, ecx      ; EBX = EBX - ECX
mul edx           ; EAX = EAX * EDX（结果在 EAX 和 EDX 中）
imul rax, rdx     ; RAX = RAX * RDX
```

### 3. 逻辑运算

```asm
and eax, ebx      ; EAX = EAX & EBX
or eax, ebx       ; EAX = EAX | EBX
xor eax, ebx      ; EAX = EAX ^ EBX
not eax           ; EAX = ~EAX
test eax, ebx     ; 对 EAX 和 EBX 按位与，影响标志位
```

### 4. 控制流

```asm
jmp label         ; 跳转到 label 标签
call function     ; 调用 function 函数
ret               ; 返回调用者
je label          ; 如果相等，跳转到 label
jz label          ; 如果零，跳转到 label
```

### 5. 比较和条件判断

```asm
cmp eax, ebx      ; 比较 EAX 和 EBX
jg label          ; 如果 EAX > EBX，跳转
jl label          ; 如果 EAX < EBX，跳转
je label          ; 如果 EAX == EBX，跳转
```

### 6. 堆栈操作

```asm
push ebp          ; 将 EBP 压入堆栈
pop ebp           ; 从堆栈弹出到 EBP
```

---

## 六、指令寻址方式（Intel 语法）

| 类型 | 示例 | 说明 |
|------|------|------|
| 立即数 | `mov eax, 5` | 直接使用常量 |
| 寄存器 | `mov eax, ebx` | 寄存器到寄存器 |
| 内存寻址 | `mov eax, [ebx + 4]` | 从 `ebx+4` 的内存地址读取 |
| 指针寻址 | `mov eax, [esi]` | 从 ESI 指向的内存地址读取 |
| 基址+偏移 | `mov eax, [ebx + esi + 4]` | 复合寻址方式 |
| 基址+索引 | `mov eax, [ebx + esi]` | 基址和索引的组合 |
| 寄存器间接 | `mov eax, [ebx]` | 从 ebx 指向的地址读取 |

---

## 七、段寄存器和内存寻址（在实模式中）

在实模式下，内存地址由段寄存器（如 `CS`、`DS`）和偏移量（如 `eax`）组合而成：

```
物理地址 = (段寄存器值 << 4) + 偏移量
```

例如：

```asm
mov ax, 0x1234
mov ds, ax
mov bx, [0x5678]    ; 等同于 mov bx, [ds:0x5678]
```

---

## 八、标志寄存器（Flags）

标志寄存器用于记录运算结果的状态信息，如零标志（ZF）、符号标志（SF）、进位标志（CF）、溢出标志（OF）等。

| 标志 | 说明 |
|------|------|
| ZF   | 零标志（结果为零） |
| SF   | 符号标志（结果为负） |
| CF   | 进位标志（无符号运算中有进位） |
| OF   | 溢出标志（有符号运算有溢出） |
| DF   | 方向标志（字符串操作方向） |
| IF   | 中断使能标志 |
| TF   | 陷阱标志（单步调试） |

---

## 九、x86-64 的扩展功能

x86-64 引入了更多寄存器和指令，支持更大的内存寻址和更复杂的操作：

### 1. 新增寄存器（Intel 语法）

- `R8-R15`：新增的通用寄存器，支持 64 位操作
- `RIP`：指令指针（类似 EIP，但 64 位）
- `RFLAGS`：标志寄存器（64 位）

### 2. 扩展指令集

x86-64 引入了 SSE、AVX、BMI、FMA 等扩展指令集，用于向量运算、位操作、浮点运算等。

---

## 十、汇编语言的常用汇编器

| 汇编器 | 语法 | 说明 |
|--------|------|------|
| NASM  | Intel 语法 | 常用于 Linux 和 Windows，支持 64 位 |
| MASM  | Intel 语法 | 微软的汇编器，用于 Windows |
| GAS   | AT&T 语法 | GNU Assembler，用于 Linux |
| FASM  | Intel 语法 | 支持 64 位，功能强大 |

---

## 十一、示例程序（Intel 语法）

### 一个简单的加法程序（在 x86-64 下）

```asm
section .text
global _start

_start:
    mov rax, 10      ; 将 10 赋值给 RAX
    mov rbx, 20      ; 将 20 赋值给 RBX
    add rax, rbx     ; RAX = RAX + RBX
    mov rdx, rax     ; 将结果保存到 RDX
    mov rdi, 1       ; 输出到 stdout
    mov rsi, msg      ; 消息地址
    mov rax, 1       ; 系统调用号 (sys_write)
    syscall

    mov rax, 60      ; 系统调用号 (sys_exit)
    xor rdi, rdi     ; 退出码为 0
    syscall

section .data
msg db 'Result: ', 0x0a, 0
```

---

## 十二、总结

| 项目 | x86 | x86-64 |
|------|-----|--------|
| 位数 | 32 位 | 64 位 |
| 寄存器数量 | 8 个通用 | 16 个通用 |
| 指令语法 | 可以是 Intel 或 AT&T | 通常使用 Intel |
| 是否支持大内存 | 无 | 支持（可寻址 48 位物理地址） |
| 是否支持 SSE/AVX | 不支持（部分支持） | 支持 |

