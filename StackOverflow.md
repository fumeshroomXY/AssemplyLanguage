# Stack Overflow
A stack overflow happens when `RSP` moves **beyond the memory region** reserved for the stack.


## Typical causes:

### Infinite / very deep recursion or Too many nested function calls
Example:
```c
void f() {
    int x;
    f();
}

//Assembly
f:
    push rbp
    mov  rbp, rsp
    sub  rsp, 4      ; local variable x

    call f           ; recursion

    leave
    ret
```
After N calls:
```c
[f frame N]
[f frame N-1]
[f frame N-2]
...
[f frame 1]
```
No base case → stack keeps growing → **stack overflow**





### Very large local variables on the stack
Example:
```c
void f() {
    char buf[1024];
}

// Assembly
push rbp
mov  rbp, rsp
sub  rsp, 1024     ; allocate 1KB local buffer
```
If this happens repeatedly (recursion), `rsp` keeps going down until:
- it hits unmapped memory → 💥 segmentation fault
- overwrites other data → undefined behavior
