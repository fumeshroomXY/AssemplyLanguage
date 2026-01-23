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
RIP = **Instruction Pointer Register**  
It holds **the address of the next instruction** the CPU will execute.

```c
ret
```

**What it does:**
- Pop return address from stack
- Jump to return address

```c
// Equivalent to:
pop RIP // restores RIP, CPU continues exactly where it left off
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

### `RBP`(saved base pointer)
The saved base pointer is:  
**The caller’s frame pointer (RBP) that a function saves on the stack when it starts.**


It lets the function:
- Build its own stack frame
- Still remember where the caller’s frame was
- Restore everything cleanly before returning

Unlike `RSP` (which moves when you push/pop),
`RBP` stays **stable** during the function.

#### Why do we “save” it?
Before the function runs, `RBP` belongs to the **caller**.

If the callee wants to use `RBP`:

- It must not destroy the caller’s value

- `RBP` is **callee-saved** by the ABI

So the function does this:
```c
push rbp        ; save caller’s RBP  ← this is the saved base pointer
mov rbp, rsp    ; establish new frame
```

### `RSP`(Stack Pointer Register)
- It holds the **memory address of the top of the stack**.
- `RSP` **moves all the time**
- On x86-64, that’s a **64-bit** address.



### Function prologue (classic)
```c
push rbp        ; save caller’s frame pointer
mov rbp, rsp    ; establish new frame
sub rsp, 32     ; allocate space for locals
```
- Creates a new stack frame
- Makes locals accessible via fixed offsets

#### What REALLY happens in function prologue
##### Initial state (before the callee starts)
- `RBP` → points to caller’s frame
- `RSP` → points to top of stack
- Stack top contains the return address (pushed by call)
```markdown
High addresses
│
│  return address   ← RSP
│
└─ lower addresses
```

##### `push rbp`
```markdown
rsp = rsp - 8        ; make space
[rsp] = rbp          ; store caller’s RBP
```
- `RSP` decreases first
- Caller’s `RBP` value is stored in stack memory

```markdown
High addresses
│
│  return address
│  saved RBP        ← RSP
│
└─ lower addresses
```

##### `mov rbp, rsp`
```markdown
rbp = rsp
```
- `RBP` points to the saved RBP slot
- This address is defined as the **base of the callee’s stack frame**

```markdown
High addresses
│
│  return address
│  saved RBP        ← RSP, RBP
│
└─ lower addresses
```

##### `sub rsp, 32`
Allocate space for local variables:
```markdown
High addresses
│
│  return address      ← [rbp + 8]
│  saved RBP           ← [rbp]
│  ------------------
│  local variables     ← [rbp - 8], [rbp - 16], ...
│
└─ rsp (moves)
Low addresses
```

##### After the prologue, the stack looks like this:
```markdown
Higher addresses
│
│  arguments (if any on stack)
│
│  return address        ← pushed by call
│  saved RBP             ← pushed by callee, "RBP" points to here
│  --------------------
│  local variables
│  temporaries
│
└── RSP (stack grows down)
Lower addresses
```
- RBP points to **saved RBP**
- `[RBP + 8]` → return address
- `[RBP - 4]` → first local variable

### Function epilogue
```c
mov rsp, rbp
pop rbp      ; restore caller’s RBP
ret

// Or the shorthand:
leave
ret
```

This ensures:
- Caller’s stack frame is intact
- Caller’s `RBP` is exactly as before

### Typical x86-64 stack frame layout
```markdown
High memory
│
│  Function arguments (if on stack)
│  -------------------------------
│  Return address      ← pushed by call
│  Saved RBP           ← old frame pointer = base anchor of the caller’s stack frame
│  -------------------------------
│  Local variables
│  Temporary space
│
└── RSP (stack grows downward)
Low memory
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

## FAQs
### Where is the size of the frame stored?
Since `RBP` stores a single address that acts as **an anchor point** inside the frame,  
Where is the size of the frame?

The answer is **the size is not needed**:

A stack frame is defined implicitly by **two moving boundaries**:
| Boundary                | Register |
| ----------------------- | -------- |
| Frame base (anchor)     | `RBP`    |
| Frame end (current top) | `RSP`    |


So conceptually:
```markdown
Frame = memory region between RBP and RSP
frame_size = RBP - RSP
```
The CPU never stores this explicitly. It’s inferred at runtime.

### Who decides how much space should be allocated for local variables?
**The compiler decides**, and the assembly just obeys.

When you write:
```c
void f() {
    int a;         // 4 bytes
    int b;         // 4 bytes
    char buf[20];  // 20 bytes
}
```
The compiler:
- Knows the size of each local variable(4 + 4 + 20 = 28 bytes)
- Knows alignment rules of the ABI(28 → 32 bytes)
- Computes a total stack frame size
- Emits assembly like: `sub rsp, 32`

Assembly itself has **no concept of “local variables”** — it only moves numbers.

#### What about variable-sized locals?
Example:
```c
void f(int n) {
    int arr[n];   // Now the size is only known at runtime.
}
```
Assembly becomes:
```c
mov eax, edi       ; n
imul eax, 4        ; n * sizeof(int)
sub rsp, rax       ; dynamic allocation
```
Now the stack pointer changes **based on a register**, not a constant.


