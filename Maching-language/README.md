# Understanding C Compilation and Assembly Code

## Overview

This guide explores what happens under the hood when you compile a C program, from source code to machine instructions, and how the CPU executes these instructions using registers and memory.

## The Compilation Process

When we compile a C program, it goes through several phases:

1. **Source Code** (.c file) → **Assembly Code** → **Object File** (via assembler)

The object file contains critical data:
- Initialized and uninitialized global values
- Literal machine instructions to execute
- Various headers, including the `.text` section with all program machine instructions

## What Are Machine Instructions?

Machine instructions aren't high-level statements like `int x = 0`. They are fundamental operations that execute very specific tasks in sequence, directly manipulating the computer's underlying hardware.

### The Low-Level Reality

While we work with high-level constructs like variables and functions, at the machine level:
- Everything is just **addresses** in the program's virtual memory during execution
- All data structures are simply **arrays of bytes**
- The CPU only works with **registers** and **memory**

## Understanding Registers

Registers are storage locations that live inside the CPU itself, making them **100-150 times faster** to access than regular memory.

### Types of Registers

1. **Integer Registers**: 16 64-bit registers used for calculations with immediate values
2. **Condition Code Registers**: 4 different flags set after every instruction executes
3. **Program Counter**: Keeps track of the current instruction's address (essential for conditional logic)

## Converting C to Assembly

You cannot directly access registers in C (unless using inline assembly). To see how registers are used, you need to examine the assembly code.

### Generating Assembly Code

```bash
gcc -O1 -S source.c
```

**Note**: The `-O1` flag is important to keep the assembly code simple and readable. Higher optimization levels (`-O2`, `-O3`) produce assembly that differs significantly from the original source code and increases compile time.

### Example

**C Code:**

<img width="283" height="200" alt="image" src="https://github.com/user-attachments/assets/570c72cb-ddc2-4107-94b2-7f5806a2efc2" />

**Generated Assembly:**

<img width="270" height="192" alt="image" src="https://github.com/user-attachments/assets/55bb9dcc-9b72-48b4-92c1-cadb695c5f56" />

## Reading Assembly Instructions

### Instruction Suffixes

Suffixes define the size of instruction operands. In computer architecture, a **word** is the smallest unit that can be transferred (16 bits or 2 bytes).

| Suffix | Size | Name |
|--------|------|------|
| `b` | 1 byte | Byte |
| `w` | 2 bytes | Word |
| `l` | 4 bytes | Long/Double Word |
| `q` | 8 bytes | Quad Word |

Each instruction manipulates either memory or registers according to these size specifications.

# Assembly Arithmetic and Control Flow

Now that we've covered registers and memory addresses, let's explore how to perform simple arithmetic operations and control program flow.

## Basic Arithmetic Operations

Assembly provides several instructions for performing arithmetic on registers. The most common operations work with quad words (64-bit values):

- `addq` - Addition
- `subq` - Subtraction
- `imulq` - Signed multiplication
- `idivq` - Signed division

<img width="784" height="532" alt="Arithmetic operations table" src="https://github.com/user-attachments/assets/1dc3c860-aa9f-4758-817c-4ec0cbe33878" />

## Bit Manipulation

Assembly includes instructions for shifting bits left and right:

**Left Shift:** Both `salq` (shift arithmetic left) and `shlq` (shift logical left) perform the same operation for left shifts.

**Right Shift:** The operation differs based on whether you need logical or arithmetic shifting:
- `shrq` - Logical right shift (fills with zeros)
- `sarq` - Arithmetic right shift (preserves sign bit)

## Moving Data

Data movement is fundamental in assembly, whether you're reading from memory to a register or storing values back to memory.

### Move Instructions

The basic move instructions are:
- `movb` - Move byte (8 bits)
- `movw` - Move word (16 bits)
- `movl` - Move long (32 bits)
- `movq` - Move quad (64 bits)

### Size Conversions

**Moving from larger to smaller:** When moving data from a larger storage location to a smaller one, the higher-order bits are truncated, and only the lower bits are retained.

**Moving from smaller to larger:** This requires explicit handling of the upper bits:
- **Sign extension** - For signed integers, preserves the sign
- **Zero extension** - For unsigned integers, fills upper bits with zeros

<img width="649" height="247" alt="Data movement instruction types" src="https://github.com/user-attachments/assets/03a780f7-fafe-4ca9-a97c-2c13cdd9a53c" />

<img width="934" height="303" alt="Sign and zero extension instructions" src="https://github.com/user-attachments/assets/dc157f35-3099-4b5c-83d9-c5da92eea7e9" />

### Important Notes

- **32-bit moves:** When moving 4 bytes into a register, the upper 32 bits of the destination register are automatically set to zero.
- **Memory operands:** Both operands of a move instruction cannot be memory locations—at least one must be a register.

## Stack Operations

- `pushq S` - Push value S onto the top of the stack
- `popq D` - Pop the top value from the stack into destination D

## Extended Arithmetic

When multiplying or dividing 8-byte values, the result may exceed 8 bytes. Special instructions handle this by using two registers:
- `%rdx` stores the upper 8 bytes
- `%rax` stores the lower 8 bytes

<img width="769" height="298" alt="Extended arithmetic instructions" src="https://github.com/user-attachments/assets/276d98b0-41bd-47a0-86dd-c2f8876456ea" />

## Condition Codes

Condition codes (flags) are crucial for conditional execution and branching. They enable the processor to make decisions based on previous operations.

### The Four Condition Flags

1. **Zero Flag (ZF)** - Set when an operation's result is zero
2. **Sign Flag (SF)** - Set when an operation's result is negative
3. **Carry Flag (CF)** - Set when unsigned arithmetic produces a carry (overflow)
4. **Overflow Flag (OF)** - Set when signed arithmetic produces overflow or underflow

### Setting Condition Codes

**Compare instruction (`cmp`):** Compares two operands by performing subtraction without storing the result. It sets the condition flags based on the comparison, primarily affecting the zero and sign flags.

**Test instruction (`test`):** Performs a bitwise AND operation on its operands without storing the result. It's commonly used with identical operands to check if a value is positive, negative, or zero.

<img width="627" height="355" alt="Compare and test instructions" src="https://github.com/user-attachments/assets/dabb3806-edea-4789-be8e-9bd0e8d29b58" />

## Jump Instructions

Jump instructions transfer control to a different part of the code. There are two types:

**Direct jumps:** Use a label (starting with `.`) to specify the target location.
```assembly
jmp .L2
```

**Indirect jumps:** Use a register or memory location (prefixed with `*`) to determine the jump target.
```assembly
jmp *8(%rax)
```

<img width="775" height="520" alt="Jump instruction variants" src="https://github.com/user-attachments/assets/cf60abce-0add-42a3-99e9-3a46713592aa" />

## Control Flow Constructs

By combining comparison and jump instructions, compilers implement familiar programming constructs such as:
- `if-else` statements
- `while` loops
- `do-while` loops
- `for` loops

When you write `if (x > 0) {...}` in a high-level language, the compiler cleverly translates it into a series of compare and conditional jump instructions.

## Conditional Moves

Conditional move instructions provide an alternative to branching that can significantly improve performance.

<img width="795" height="451" alt="Conditional move instructions" src="https://github.com/user-attachments/assets/7f10bbdf-6ef9-479a-aca4-84ea4e79d413" />

### Why Use Conditional Moves?

Modern processors use **branch prediction** to speculatively execute instructions before knowing whether a conditional branch will be taken. This keeps the processor pipeline full and maximizes performance. However, when the prediction is incorrect, all speculative work must be discarded, resulting in significant performance penalties (pipeline stalls).

Conditional moves avoid this problem by:
1. Computing results for both branches
2. Selecting the correct result based on the condition
3. Eliminating the need for branch prediction

This approach is particularly effective when:
- Both computations are simple and quick
- The conditional code is frequently executed
- Branch prediction would be unreliable

The compiler can generate more efficient code by using conditional moves instead of conditional branches, maintaining a full instruction pipeline without the risk of misprediction penalties.
