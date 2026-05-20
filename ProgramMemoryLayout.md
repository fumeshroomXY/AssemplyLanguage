# Typical program memory layout
```markdown
High address
│
│   Stack
│   ↓ grows downward
│
│   Heap
│   ↑ grows upward
│
│   Static / Global data
│   (data, bss)
│
│   Literals (read-only data)
│
│   Instructions (text / code)
│
Low address
```

## Program stack (Stack)
- Manages **function calls**
- Stores return addresses, function parameters (sometimes), local variables, saved registers
- Limited size

### Function Call
When a function call happens, all stack frames are created on the program stack (**the stack region**).
```markdown
Higher addresses
│
│  caller's frame
├────────────────
│  return address
│  saved RBP      ← RBP
│  local vars
│  temp values    ← RSP
│
Lower addresses
```

```markdown
Memory region: STACK

High address
┌────────────────────────┐
│ frame of main()        │
├────────────────────────┤
│ frame of foo()         │  ← created on function call
├────────────────────────┤
│ frame of bar()         │
└────────────────────────┘
Low address
```



## Heap
- Dynamic memory allocation
- Stores memory allocated via `malloc`, `calloc`, `new`
- Slower than stack
- Large

## Static / Global data
- Exists for the **entire program lifetime**.
- Allocated at program start, freed at program termination

### `.data` (initialized globals/statics)
```c
int g = 5;
static int s = 10;
```

### `.bss` (uninitialized globals/statics)
```c
int g;
static int s;
```

## Literals (Read-only data)
- Stores constants that should not change
- Attempting to modify → undefined behavior / crash

```c
"Hello world"     // string literal
const int x = 42; // often placed here
```

## Instructions (Text/Code segment)
- Stores **machine instructions** (compiled code)
- Every instruction the CPU executes is fetched from the instruction (code / text) segment
- CPU updates `RIP`(instruction pointer) to the next instruction after executing an instruction
- Stack contains only data like return addresses and locals, not the code

### Operations on `RIP` are mostly *hidden* in assembly language.

Example:
```markdown
add rax, rbx
mov rcx, rdx
```
What happens:

- CPU fetches instruction at `RIP`
- Executes it
- Automatically advances `RIP` to the next instruction


This update is **implicit and invisible** in assembly syntax.


Assembly shows **control flow intent**, not micro-operations.

## How much space should be allocated for every part

| Memory part            | Who decides size?         | When?        |
| ---------------------- | ------------------------- | ------------ |
| Instructions (.text)   | Compiler + linker         | Compile/link |
| Literals (.rodata)     | Compiler + linker         | Compile/link |
| Static/global (.data)  | Compiler + linker         | Compile/link |
| Static/global (.bss)   | Compiler + linker         | Compile/link |
| Stack frame (per func) | **Compiler**              | Compile time |
| Stack (total size)     | OS / runtime              | Program load |
| Heap (total size)      | OS + allocator            | Runtime      |
| Heap allocation size   | **Programmer** (`malloc`) | Runtime      |

### Instructions (.text)
- Compiler translates your source code → machine instructions
- Each instruction has a **known size**
- Total size = sum of instruction bytes

### Literals (.rodata)
- Literals are known at compile time

```c
"Hello"  // 6 bytes containing '\0'
const int x = 42;
```

### Static / global data (.data / .bss)
- Compiler knows type → int → 4 bytes
```c
int g = 10;
int g;
```

### Stack frames (important)
- Everything in a stack frame has a **compile-time size**.

Example:
```c
void f() {
    int a;        // 4 bytes
    double b;     // 8 bytes
    char buf[16]; // 16 bytes
}

// 4 + 8 + 16 = 28
// + padding (alignment)
// = 32 bytes

sub rsp, 32
```

Stack frame size is:
- Per-function
- Fixed at compile time
- Encoded directly into instructions

#### What if there’s recursion?
**Same size per call**. Each call gets its own frame.

#### What if I have an array whose size is not fixed?
##### In C (before C99): size must be compile‑time constant
```c
int n = input();
int arr[n];   // ❌ illegal in C89
```

##### In C99 and later: VLAs (Variable Length Arrays)
C99 introduced **variable-length arrays**, which do allow runtime-sized arrays on the stack:
```c
int n = input();
int arr[n];   // ✔ legal in C99
```
The compiler generates dynamic stack adjustment:
```markdown
mov eax, n
imul eax, 4
sub rsp, rax
```
So the stack frame size is **not fixed** anymore — it becomes dynamic.

But this has consequences:
- You cannot take `sizeof(arr)` at compile time.
- Debuggers and static analyzers hate it.
- Many compilers disable VLAs by default.


##### In C++: VLAs are NOT allowed
C++ requires all stack object sizes to be known at compile time.

So this is illegal:
```c
int n;
cin >> n;
int arr[n];   // ❌ not allowed in C++

// Instead you must use:
std::vector<int> (heap)
new int[n] (heap)
std::unique_ptr<int[]> (heap)
```

### The stack (total stack size)
This is decided by: OS, Program loader and Runtime environment.

### Heap
Heap grows at runtime, managed by the OS + allocator (`malloc`)
