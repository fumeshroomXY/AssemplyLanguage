# Segmentation Fault
A segmentation fault happens when the CPU tries to **access memory it is not allowed to access**.

## The chain of events (important)
```css
C code
 ↓
assembly generates a memory address
 ↓
CPU tries to load/store from that address
 ↓
MMU checks permissions
 ↓
❌ illegal → SIGSEGV
```
So the crash happens at the memory access instruction.

## Common Causes
### Classic: NULL pointer dereference
```c
int *p = NULL;
*p = 5;

mov rcx, 0
mov DWORD PTR [rcx], 5   ; 💥
```
- Address `0x0` is **never mapped**

### Uninitialized pointer
```c
int *p;
*p = 10;

; p contains random stack garbage
mov rcx, [rbp-8]
mov DWORD PTR [rcx], 10   ; 💥 maybe
```

Why it’s dangerous:
- Sometimes crashes
- Sometimes corrupts memory
- Sometimes “works”

### Array out of bounds
```c
int arr[4];
arr[100] = 1;

mov DWORD PTR [rbp-32 + 100*4], 1
```

- Compiler does **no bounds check**, just computes an address

If that address:
- is still inside stack → silent corruption
- crosses into unmapped memory → 💥

### Stack overflow (recursion)
```c
void f() {
    int a[1000];
    f();
}

sub rsp, 4000
call f
```
Eventually:
- `rsp` moves into unmapped memory
- next stack write faults

### Use-after-free (heap)
```c
int *p = malloc(4);
free(p);
*p = 10;

mov DWORD PTR [rax], 10   ; 💥 or silent corruption
```
Why it’s unpredictable:
- memory may still be mapped
- allocator may reuse it
- OS may protect it

### Writing to read-only memory
```c
char *s = "hello";
s[0] = 'H';

mov BYTE PTR [rax], 'H'   ; 💥
```
- string literals live in **read-only (.rodata)**
- write access is forbidden

## Why it’s called segmentation fault
Historically:
- Memory divided into **segments**
- Access outside segment → fault
