# Memory Leak
Happens when a program allocates memory but **never releases it**

## Why it happens
- Forgetting to free memory (C / C++)
```c
int *p = malloc(100 * sizeof(int));
// use p
// missing: free(p);
```

- Losing the pointer
```c
int *p = malloc(100);
p = malloc(200);   // original 100 bytes are leaked
```

- Early return or error path
```c
int *p = malloc(100);
if (error)
    return;        // memory is never freed
free(p);
```

## Effects of memory leaks
- Increasing memory usage over time
- Slower performance
- Program crashes
- System out-of-memory errors

## How to prevent memory leaks
- Always pair malloc with free, new with delete
- Use **RAII** (smart pointers like std::unique_ptr)
- Use tools like **Valgrind, AddressSanitizer**
