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

## 熟悉gdb工具
建议事先阅读CSAPP中的3.10.2章节，这里给出部分指令参考：

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
| x           | examine          | 打印内存中的值                                   |
| j           | jump             | 跳转到程序指定位置                               |
| layout asm  | assembly layout  | 显示汇编代码视图                                 |
| layout regs | register layout  | 显示当前的寄存器状态和它们的值                   |

## 了解炸弹结构
首先，查看一下给出的 `bomb.c` 文件，了解炸弹的整体结构。
```C
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
# 设置断点
前文提到，我们不希望程序运行 `explode_bomb` 函数，因此我们利用 `gdb` 工具给 `explode_bomb` 设置断点，防止程序继续执行到引爆炸弹的指令。
```bash
gdb ./bomb
(gdb) b explode_bomb
```
# Phase 1
# Phase 2
# Phase 3
# Phase 4
# Phase 5
# Phase 6
# Secret Phase