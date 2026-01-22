# Function call at the assembly level

A function call is basically:  
“Jump to another piece of code, remember where to come back, pass inputs, then come back with a result.”

At the assembly level, that means four things must happen:

- **Arguments are passed** (registers and/or stack)
- **Return address** is saved
- Control **jumps** to the function
- Function **returns**, restoring control to the caller

## How are parameters passed? (Registers vs Stack)
This depends on the **calling convention**.  

**x86-64 System V ABI (Linux / macOS) — most common**


### Integer / pointer arguments:
| Argument | Register |
| -------- | -------- |
| 1st      | `RDI`    |
| 2nd      | `RSI`    |
| 3rd      | `RDX`    |
| 4th      | `RCX`    |
| 5th      | `R8`     |
| 6th      | `R9`     |

### Floating-point arguments:
- Passed in `XMM0–XMM7`

### More than 6 arguments?
- Spilled onto the stack  
Example:

```c
foo(a, b, c, d, e, f, g);
```
- `a–f` → registers
- `g` → stack


## Where is the return value stored?
Again, defined by the **calling convention**.
### x86-64
| Return type       | Location                       |
| ----------------- | ------------------------------ |
| Integer / pointer | `RAX`                          |
| Floating point    | `XMM0`                         |
| Small struct      | `RAX` / `RDX`                  |
| Large struct      | Caller provides hidden pointer |

Example:
```c
call add
; result now in RAX
```

## What exactly do `call` and `ret` do?(extremely important)
```c
call foo
```
**What happens internally:**
- Push return address onto the stack
- Jump to function address

```c
// Equivalent to:
push RIP   ; address of next instruction
jmp foo
```  


```c
ret
```

**What it does:**
- Pop return address from stack
- Jump to return address

```c
// Equivalent to:
pop RIP
```
That’s why **stack corruption causes crashes** — `ret` jumps to garbage.

## Stack Frame
A stack frame is the **chunk of stack memory** used by **one function call**.  
It typically contains:
- Saved base pointer
- Return address
- Local variables
- Saved registers
- Stack-passed arguments

### Typical x86-64 stack frame layout
```markdown
High memory
│
│  Function arguments (if on stack)
│  -------------------------------
│  Return address      ← pushed by call
│  Saved RBP           ← old frame pointer
│  -------------------------------
│  Local variables
│  Temporary space
│
└── RSP (stack grows downward)
Low memory
```
### Function prologue (classic)
```c
push rbp        ; save caller’s frame pointer
mov rbp, rsp    ; establish new frame
sub rsp, 32     ; allocate space for locals
```
- Creates a new stack frame
- Makes locals accessible via fixed offsets

### Function epilogue
```c
mov rsp, rbp
pop rbp
ret

// Or the shorthand:
leave
ret
```

## Complete example (C → Assembly)
**C Code**
```c
int add(int a, int b) {
    return a + b;
}
```

**x86-64 Assembly (simplified)**
```c
add:
    push rbp
    mov rbp, rsp

    mov eax, edi    ; a
    add eax, esi    ; b

    pop rbp
    ret
```

**Call site**
```c
mov edi, 2
mov esi, 3
call add
; result in eax
```

