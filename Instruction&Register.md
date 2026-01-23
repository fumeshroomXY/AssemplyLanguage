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

