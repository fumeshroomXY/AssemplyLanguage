# Instructions in Assembly Language

## `mov`
```markdown
mov destination, source
mov rbp, rsp
```
means: **Copy the value of `RSP` into `RBP`**

## `push` and `pop`
`push` and `pop` **move values between registers and memory via the stack**.

### What does `push` actually do?
Example:
```c
push rax

// means:
rsp = rsp - 8        ; make room (x86-64)
[rsp] = rax          ; store the VALUE of rax in memory
```

- `RSP` moves
- The value in `RAX` is written into memory
- Register `RAX` itself is **unchanged**.

### What does `pop` actually do?
Example:
```c
pop rax

// means:
rax = [rsp]          ; load VALUE from memory
rsp = rsp + 8        ; reclaim space
```
- Memory → register
- Stack pointer moves back up

# Assembly Control Flow
## `cmp`
```c
cmp a, b // a - b
```
`cmp` never changes registers, only **updates CPU flags**:
- ZF (Zero Flag) → result == 0
- SF (Sign Flag) → result < 0
- OF (Overflow Flag) → signed overflow

## Conditional jumps (`je`, `jne`, `jg`, `jl`)
These **depend on flags** set by `cmp`.
| Assembly | Meaning                  | C equivalent |
| -------- | ------------------------ | ------------ |
| `je`     | jump if equal            | `==`         |
| `jne`    | jump if not equal        | `!=`         |
| `jg`     | jump if greater (signed) | `>`          |
| `jge`     | jump if greater or equal (signed) | `>`          |
| `jl`     | jump if less (signed)    | `<`          |

`jg` / `jl` are signed comparisons (there are also `ja` / `jb` for unsigned)

Example:
```c
if (a == b) {
    do_something();
}

// Assembly
cmp a, b          // set flag
jne end_if      ; // jump if flag is false
call do_something
end_if:
```

## `jmp`(unconditional jump)
Always jumps. **No condition. No flags.**
```c
jmp label

// C equivalent
goto label;
```

Example:
```c
if (a > b) {
    x = 1;
} else {
    x = 2;
}

// Assembly
cmp a, b
jle else_part   ; if a <= b → else
mov x, 1
jmp end_if

else_part:
mov x, 2

end_if:
```

## Loops
### `while` loop
Example:
```c
while (i < 10) {
    i++;
}

// Assembly
loop_start:
cmp i, 10
jge loop_end    ; if i >= 10, exit loop

inc i
jmp loop_start

loop_end:
```

### `for` loop
Example:
```c
for (int i = 0; i < 5; i++) {
    sum += i;
}

// Assembly
mov i, 0          ; init

for_start:
cmp i, 5          ; condition
jge for_end       ; if i >= 5 → exit

add sum, i        ; body

inc i             ; update
jmp for_start     ; repeat

for_end:
```
