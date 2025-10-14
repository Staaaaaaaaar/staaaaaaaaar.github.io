---
title: ICS | 拆弹！我吗？
date: 2025-10-14 10:08:28
categories:
  - NOTE
tags:
  - pku
  - ics
excerpt: 北京大学2025年秋季学期计算机系统导论Bomb Lab记录。
---

# 前置准备

## 熟悉 GDB 工具

建议事先阅读 CSAPP 中的 3.10.2 章节，这里给出部分指令参考：

| 指令        | 含义             | 描述                                             |
| ----------- | ---------------- | ------------------------------------------------ |
| r           | run              | 开始执行程序，直到下一个断点或程序结束           |
| b           | break            | 在指定位置设置断点                               |
| q           | quit             | 退出 GDB 调试器                                  |
| ni          | next instruction | 执行下一条指令                                   |
| si          | step instruction | 执行当前指令，若是函数调用则进入函数             |
| n           | next             | 执行下一行代码                                   |
| s           | step             | 执行下一行代码，若有函数调用则进入函数           |
| c           | continue         | 从当前位置继续执行程序，直到下一个断点或程序结束 |
| p           | print            | 打印变量的值                                     |
| p $rax      |                  | 打印 %rax 寄存器的值                             |
| p/x $rsp    |                  | 以十六进制打印 %rsp 寄存器的值                   |
| p/d $rsp    |                  | 以十进制打印 %rsp 寄存器的值                     |
| x           | examine          | 打印内存中的值                                   |
| x/2x $rsp   |                  | 以十六进制格式查看 %rsp 处的 2 个内存单元        |
| x/2c $rsp   |                  | 以字符格式查看 %rsp 处的 2 个内存单元            |
| x/s $rsp    |                  | 将 %rsp 处的内存视为 C 风格字符串查看            |
| x/b $rsp    |                  | 查看 %rsp 处的 1 个字节内存                      |
| x/h $rsp    |                  | 查看 %rsp 处的 1 个字（2 字节）内存              |
| x/w $rsp    |                  | 查看 %rsp 处的 1 个二字（4 字节）内存            |
| x/g $rsp    |                  | 查看 %rsp 处的 1 个四字（8 字节）内存            |
| j           | jump             | 跳转到程序指定位置                               |
| layout asm  | assembly layout  | 显示汇编代码视图                                 |
| layout regs | register layout  | 显示寄存器状态视图                               |

## 了解炸弹结构

首先，查看一下给出的 `bomb.c` 文件，了解炸弹的整体结构。

```c
int main(int argc, char *argv[])
{
    // ...
    input = read_line();
    phase_1(input);
    phase_defused(fp);
    printf("Phase 1 defused. How about the next one?\n");
    // ...
    return 0;
}
```

炸弹程序包含六个 `phase` ，每个 `phase` 都会读入一行输入，然后调用一个 `phase_defused` 函数来验证输入。

接着，我们反汇编 `bomb` 可执行文件，查看每个炸弹的细节。

```bash
objdump -d bomb > bomb.asm
```

阅读汇编代码，我们发现每个 `phase` 函数都包含一个 `explode_bomb` 的调用指令，形如：

```x86asm
27c1:   e8 1d 08 00 00       	call   2fe3 <explode_bomb>
```

显而易见，这个函数会在输入错误时引爆炸弹，这是我们不想看到的。

## 基本配置

### 设置断点

前文提到，我们不希望程序运行 `explode_bomb` 函数，因此我们利用 `gdb` 工具给 `explode_bomb` 设置断点，防止程序继续执行到引爆炸弹的指令。

```bash
gdb bomb
(gdb) b explode_bomb
```
同时，为了方便我们对每个 `phase` 进行调试，我们也可以在每个 `phase` 函数入口处设置断点。

```bash
(gdb) b phase_1
(gdb) b phase_2
(gdb) b phase_3
(gdb) b phase_4
(gdb) b phase_5
(gdb) b phase_6
```
### 打开视图
为了更好地观察程序每一步的执行情况以及寄存器状态，我们可以打开汇编代码视图和寄存器状态视图。

```bash
(gdb) layout asm
(gdb) layout regs
```
### 输入重定向

查看原代码发现，`bomb.c` 程序提供了文件读入的方式。为方便调试，我们可以将输入重定向到一个文件 `psol.txt` 中。

```bash
gdb bomb psol.txt
```

### 添加默认配置

为了避免每次启动 `gdb` 都要手动设置断点，我们可以在当前目录下创建一个 `.gdbinit` 文件来设置 `gdb` 进入时的一些默认配置。
```bash
# 创建当前目录下的 .gdbinit 文件
touch .gdbinit
# 创建 .config/gdb 文件夹
mkdir -p ~/.config/gdb
# 允许 gdb 预加载根目录下所有的文件
echo "set auto-load safe-path /" > ~/.config/gdb/gdbinit
```
在 `.gdbinit` 中添加默认配置：
```
set args psol.txt

layout asm
layout regs

b explode_bomb

b phase_1
b phase_2
b phase_3
b phase_4
b phase_5
b phase_6

r
```

完成这些之后，我们就可以直接运行 `gdb bomb` 来开始拆弹了。
# Phase 1

# Phase 2

# Phase 3

# Phase 4

# Phase 5

# Phase 6

```c
int main() {
    // 初始化变量，对应原代码中的寄存器
    int ax, bx, cx, dx, bp, sp, si;

    // 代码片段1
    sp = 0;
    bp = 0;

    // 标号3处的代码
    while (bp <= 5) {
        ax = bp;
        ax = 2 + (4 * ax) - 2;

        if (ax <= 5) {
            dx = bp + 2; // 推测原代码含义
        }

        bx = bx + 5;
    }

    // 标号6处的代码
    while (ax == bp) {
        ax = bx;
        dx = 7 + (4 * ax);

        if (dx == 7 + (4 * ax)) {
            // 原代码中的"break?"推测为break
            break;
        } else {
            bx++;
        }

        bp = dx;
    }

    // 标号10处的代码
    ax = 0;

    // 标号14处的代码
    while (ax <= 5) {
        cx = ax;
        dx = 7 - (4 * cx);
        memory[4 * ax] = dx; // 处理[4*ax] = dx
        ax++;
    }

    // 标号19处的代码
    si = 0;
    while (si <= 5) {
        ax = 1;
        dx = /* 原代码"rip + 0x6b4a"难以直接转换，这里做简化 */ 0;
        dx = si;

        // 标号55处的代码
        while (ax < (4 * cx)) {
            dx = dx + 8;
            ax++;
            ax = si;
            memory[8 * cx + 8] = dx; // 处理[8*cx+8] = dx
        }
    }

    // 右侧代码片段
    // 标号28处的代码
    while (bp <= 4) {
        ax = bx + 8;
        ax = /* 原代码"[bx]"推测为数组访问 */ memory[bx];

        if (ax < /* 原代码"[bx]" */ memory[bx]) {
            // 原代码"bomb?"推测为某种处理
            printf("触发条件: bomb?\n");
        } else {
            bx = /* 原代码"[bx + 2]" */ memory[bx + 2];
        }
        bp++;
    }

    // 其他零散代码
    ax = sp + 2; // 处理[sp + 2]

    // 标号26处的代码
    while (ax < 5) {
        dx = ax;
        dx = 18 + dx + 32; // 处理[8*dx+32]
        memory[cx + 7] = dx;
        ax++;
        cx = dx;
        ax = dx;
    }

    memory[cx + 8] = 0;
    bp = 0;

    return 0;
}
```

# Secret Phase
