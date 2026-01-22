Example: **x86-64 System V ABI (Linux / macOS)**
- x86-64, CPU architecture; x86, instruction set; 64-bit mode
- System V ABI, ABI rule

# ABI (Application Binary Interface)
An ABI is a **contract** between:
- The compiler
- The OS
- The CPU
- Other compiled code (libraries, object files)


The ABI lives at the **boundary between user programs and the OS**.  

ABI depends on three layers：
```markdown
CPU architecture
        +
Operating system conventions
        +
Compiler / toolchain agreement
```




ABI defines **binary-level rules**, such as:

**Function call rules**
- Which registers pass arguments
- Where return values go
- Who saves which registers

**Memory layout rules**
- Stack growth direction
- Stack alignment (16-byte on x86-64!)
- Stack frame layout

**Data representation**
- Sizes of int, long, pointers
- Struct layout and padding
- Endianness (little-endian)

**Object / executable format**
- ELF on Linux
- Mach-O on macOS

In practical terms: GCC, Clang, and system libraries **all agree on**:
- Argument registers (`RDI`, `RSI`, …)
- Return register (`RAX`)
- Stack alignment
- Which registers must be preserved

That’s why:
- Code compiled by GCC can call code compiled by Clang
- Your program can call `printf`
- Assembly and C can interoperate safely

# Why this matters (very important)
**Same CPU ≠ same ABI**  
Example:
| OS      | ABI                   |
| ------- | --------------------- |
| Linux   | x86-64 System V ABI   |
| macOS   | x86-64 System V ABI   |
| Windows | **Microsoft x64 ABI** |
Same machine, different rules.  

```c
foo(a, b, c, d);
```
| ABI         | First argument |
| ----------- | -------------- |
| System V    | `RDI`          |
| Windows x64 | `RCX`          |


If you mix them → **instant crash**

# Concrete example
**C code**
```c
int sum(int a, int b);
```
**x86-64 System V ABI guarantees:**
- `a` → `EDI`
- `b` → `ESI`
- Return → `EAX`


So assembly must follow:
```c
mov edi, 3
mov esi, 4
call sum
; result in eax
```

# FAQs
## Is ABI different when the OS is different?
Usually YES. Even on the same CPU, different OSes choose different rules.  
Example: x86-64 CPU
| OS      | ABI                 |
| ------- | ------------------- |
| Linux   | x86-64 System V ABI |
| macOS   | x86-64 System V ABI |
| Windows | Microsoft x64 ABI   |

## Why do Linux and macOS share “System V”?
Because both are:
- UNIX-like
- Descended from (or compatible with) System V UNIX ideas
- Designed to share tooling and libraries

They chose to follow the **System V AMD64 ABI spec**.

Important nuance:
- macOS uses Mach-O binaries
- Linux uses ELF
- But function calling rules are the same

So: Same ABI rules for function calls, different executable formats

## Can the same OS have multiple ABIs?
**YES** (very important) 

**Linux**:
| Architecture | ABI            |
| ------------ | -------------- |
| x86 (32-bit) | i386 SysV ABI  |
| x86-64       | SysV AMD64 ABI |
| ARM32        | AAPCS          |
| ARM64        | AAPCS64        |

## Can the same OS + CPU have multiple ABIs?
Yes.


**Linux x86-64**:
- System V ABI (native)
- x32 ABI (64-bit registers, 32-bit pointers)
- i386 ABI (via compatibility mode)

