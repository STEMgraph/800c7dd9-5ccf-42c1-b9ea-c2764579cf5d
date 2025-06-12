<!---
{
  "id": "800c7dd9-5ccf-42c1-b9ea-c2764579cf5d",
  "depends_on": ["718193ef-11a1-408d-af23-4b10c24d490d"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-06-12",
  "keywords": ["assembly", "GAS", "jmp", "cmp", "conditional", "Linux"]
}
--->

# Control Flow with `jmp` and `cmp` in GAS

> In this exercise you will learn how to use unconditional and conditional jumps in x86\_64 assembly using GAS syntax. Furthermore we will explore how the `cmp` instruction interacts with conditional jumps to enable basic decision-making in assembly code.

## Introduction

Until now, you've written **linear programs** — a sequence of instructions executed one after another from top to bottom. But how does the CPU know which instruction to run next? This is where the **program counter (PC)** comes in. The program counter is a special register in the processor that always holds the **memory address of the next instruction to execute**. Every time the CPU finishes executing an instruction, it looks at the PC to decide what to do next, then usually increments it to point to the following instruction.

To implement **data-dependent program flow** — where the next action depends on a variable or condition — we must **manipulate the program counter**. This is done using jump instructions like `jmp`, `je`, and `jl`. These instructions don't just compute values; they **redirect the program counter**, altering the order of execution based on logic or data. In essence, they allow your programs to make decisions and repeat actions, transforming static instruction lists into dynamic logic circuits.

To understand how conditional jumps work, we must introduce another critical concept in CPU architecture: **flags**.

A **flag** is a special single-bit indicator stored in a special-purpose register, commonly called the **FLAGS** or **EFLAGS** register. After certain instructions — most notably arithmetic and comparison operations — these flags are updated to reflect the result. For example:

* The **Zero Flag (ZF)** is set if the result of an operation is zero.
* The **Sign Flag (SF)** indicates if the result was negative.
* The **Overflow Flag (OF)** flags arithmetic overflows.
* The **Carry Flag (CF)** signals a carry out of the most significant bit.

When we use the `cmp` instruction, it performs a subtraction internally and sets these flags without storing the result. Jump instructions like `je` (jump if equal) or `jl` (jump if less) then **check these flags** to determine if they should redirect the program counter.

This mechanism lets us implement **conditional behavior** — that is, to do one thing or another based on some value. In higher-level languages like C or Python, this is done using an `if` statement:

```c
if (x < y) {
    // do something
}
```

In assembly, the same logic is achieved by comparing `x` and `y`, and then jumping conditionally based on the flags that result from that comparison.

Thus, while you won't see an `if` keyword in assembly, the underlying logic — conditional branching — is made possible through **flags and conditional jumps**. These are implemented as circuits in the processor that control the program counter. They are essential for building any kind of decision-making into your programs, enabling not just branching, but also loops, retries, and state machines.

At the hardware level, instructions like `jmp`, `je`, and `jl` are more than just logical checks. These are implemented as dedicated circuits in the processor's control unit. Unlike arithmetic instructions that operate on data and store results in registers or RAM, jump instructions **manipulate the program counter (PC)** — the register that tells the CPU which instruction to execute next. This means a successful conditional jump doesn't just compute a value, it directly redirects the CPU to another point in the code. These jumps are thus fundamental mechanisms for branching, decision-making, and looping, tightly integrated into the architecture of the processor itself.

Having previously worked with direct syscalls for terminal output, we now explore **control flow**, the bedrock of program logic. In x86\_64 assembly, the ability to change the flow of execution is managed using jump instructions. These are analogous to `if`, `while`, or `goto` in high-level languages.

The `jmp` instruction provides an **unconditional jump** to a specified label. Execution simply continues from the target location. On the other hand, **conditional jumps** such as `je` (jump if equal) or `jne` (jump if not equal) depend on **flags** set by prior comparison operations.

The typical setup for conditional logic involves:

1. **Comparing values** using the `cmp` instruction.
2. **Jumping conditionally** based on the flags set by `cmp`.

For example, to print different messages based on a value, we compare registers and choose a label to jump to, just like a rudimentary `if` statement.

```gas
    mov $3, %rax
    cmp $5, %rax     # Compares 5 with %rax (sets flags)
    jl less_than     # Jump if %rax < 5
    jg greater_than  # Jump if %rax > 5
```

These instructions allow us to build loops, branches, and full logic trees — all in pure assembly.

### Further Readings and Other Sources

* [https://en.wikibooks.org/wiki/X86\_Assembly/Control\_Flow](https://en.wikibooks.org/wiki/X86_Assembly/Control_Flow)
* [https://blog.rchapman.org/posts/Linux\_System\_Call\_Table\_for\_x86\_64/](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)
* [https://www.cs.virginia.edu/\~evans/cs216/guides/x86.html](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html)
* [YouTube: "Assembly Tutorial 6 - Conditional Branches"](https://www.youtube.com/watch?v=fgoWNwMFQ7w)

## Tasks

### 1. Basic `jmp`

Create a minimal program that prints:

```
This will always be printed.
This will be skipped.
```

Use a `jmp` instruction to skip the second line.

```gas
.section .data
msg1: .ascii "This will always be printed.\n"
len1 = . - msg1
msg2: .ascii "This will be skipped.\n"
len2 = . - msg2

.section .text
.globl _start

_start:
    # Print first message
    mov $1, %rax
    mov $1, %rdi
    lea msg1(%rip), %rsi
    mov $len1, %rdx
    syscall

    jmp end

    # Print second message
    lea msg2(%rip), %rsi
    mov $len2, %rdx
    syscall

end:
    mov $60, %rax
    xor %rdi, %rdi
    syscall
```

Add this code to a fresh file:
```sh
vim assembly-file.s
```

To compile, use the instructions that you already know from earlier exercises:
```sh
as -o object-file.o assembly-file.s
ld -o output-binary object-file.o
./output-binary
```

### 2. Using `cmp` and `je`

Compare two values using `cmp`. If equal, jump to a label that prints:

```
Equal!
```

Otherwise, print:

```
Not equal!
```

```gas
.section .data
equal_msg: .ascii "Equal!\n"
equal_len = . - equal_msg
notequal_msg: .ascii "Not equal!\n"
notequal_len = . - notequal_msg

.section .text
.globl _start

_start:
    mov $5, %rax
    mov $5, %rbx
    cmp %rbx, %rax
    je equal

    lea notequal_msg(%rip), %rsi
    mov $notequal_len, %rdx
    jmp print

equal:
    lea equal_msg(%rip), %rsi
    mov $equal_len, %rdx

print:
    mov $1, %rax
    mov $1, %rdi
    syscall

    mov $60, %rax
    xor %rdi, %rdi
    syscall
```

### 3. Branching Example

Use `cmp` followed by:

* `jl` for "less than"
* `je` for "equal"
* `jg` for "greater than"

Print appropriate messages:

* "X is less than Y"
* "X equals Y"
* "X is greater than Y"

```gas
.section .data
lt_msg: .ascii "X is less than Y\n"
lt_len = . - lt_msg
eq_msg: .ascii "X equals Y\n"
eq_len = . - eq_msg
gt_msg: .ascii "X is greater than Y\n"
gt_len = . - gt_msg

.section .text
.globl _start

_start:
    mov $4, %rax
    mov $3, %rbx
    cmp %rbx, %rax
    jl less
    je equal
    jg greater

less:
    lea lt_msg(%rip), %rsi
    mov $lt_len, %rdx
    jmp print

equal:
    lea eq_msg(%rip), %rsi
    mov $eq_len, %rdx
    jmp print

greater:
    lea gt_msg(%rip), %rsi
    mov $gt_len, %rdx

print:
    mov $1, %rax
    mov $1, %rdi
    syscall

    mov $60, %rax
    xor %rdi, %rdi
    syscall
```

### 4. Loop with Decrement

Create a loop that prints numbers from 3 to 1 using `cmp`, `dec`, and `jne`.
Use syscall to print numbers — hardcode them as strings in the `.data` section.

```gas
.section .data
n3: .ascii "3\n"
l3 = . - n3
n2: .ascii "2\n"
l2 = . - n2
n1: .ascii "1\n"
l1 = . - n1

.section .text
.globl _start

_start:
    mov $3, %rcx

loop:
    cmp $3, %rcx
    je print3
    cmp $2, %rcx
    je print2
    cmp $1, %rcx
    je print1
    jmp end

print3:
    lea n3(%rip), %rsi
    mov $l3, %rdx
    jmp prn

print2:
    lea n2(%rip), %rsi
    mov $l2, %rdx
    jmp prn

print1:
    lea n1(%rip), %rsi
    mov $l1, %rdx

prn:
    mov $1, %rax
    mov $1, %rdi
    syscall
    dec %rcx
    cmp $0, %rcx
    jne loop

end:
    mov $60, %rax
    xor %rdi, %rdi
    syscall
```

## Questions

1. What flags are affected by the `cmp` instruction?
2. How does `cmp` differ from `sub` in behavior?
3. Why must jump labels be local (and defined before use)?
4. What would happen if you used `jmp` to jump into the middle of a syscall?

## Advice

Jumping and comparing are the assembly equivalents of logic and conditionals — foundational for real program control. By experimenting with `jmp`, `cmp`, and their conditional variants, you are laying the groundwork for understanding loops, decision trees, and even finite state machines in bare metal code. Stick with it, and remember that this control flow mastery will unlock all kinds of patterns in subsequent exercises like [working with loops](#) or [implementing functions](#).
