# Pointers, Shellcode and Syscalls

## Pointer Types

A **pointer type** holds a memory address instead of a direct value, and specifies the type of what lives at that address:

- **`uint8_t *`**: Pointer to a byte (`uint8_t`).
- **`uint8_t (*)[0x1000]`**: Pointer to an entire 4096-byte array (`&shellcode_buffer` has this type).
- **`void *`**: Pointer to raw, untyped memory ("address only, no assumptions about what type is stored there").
- **`void (*)()`**: Pointer to a function returning `void`. Notice the parentheses around `(*)`—without them, `void *f()` would mean a *function* returning a `void *` pointer, whereas `void (*f)()` means a *pointer to a function*.

Your buffer might start as `uint8_t shellcode_buffer[0x1000]` (type `uint8_t (*)[0x1000]`), or come from `malloc()` / `mmap()` (which already returns `void *`). Casting to `(void *)` normalizes it to a plain "memory location" pointer.

## Function Pointers

The empty parentheses in `void(*)()` declare a pointer to a function returning `void` with **unspecified parameters**.

**Reinterpret the address as code and call it**: `((void(*)())shellcode)();` takes the numeric address inside `shellcode`, tells the compiler "treat this address as the entry point of a function," and emits an indirect call instruction (such as `call rax`) so the CPU's instruction pointer (`rip` / `pc`) jumps to that address and begins fetching and executing those bytes as instructions.

```c
void (*func)();
```

This declares `func` as a pointer to a function that returns `void`. The `()` means its arguments are unspecified in this C declaration (rather than explicitly declaring zero arguments).

**The `()` at the very end** in `((void(*)())shellcode)();` is the **call operator** ("run it right now with 0 arguments").

`shellcode` is always a **location (address)**, and the CPU goes to that address to read the **machine code**.

The compiler generates an indirect call instruction, conceptually:

```asm
call rax
```

where `RAX` contains the address of your shellcode.

## Syscalls

On x86-64 Linux, the `syscall` instruction requests a service from the kernel.

The syscall convention is:

| Register | Purpose |
|---|---|
| `RAX` | System call number |
| `RDI` | Argument 1 |
| `RSI` | Argument 2 |
| `RDX` | Argument 3 |
| `R10` | Argument 4 |
| `R8` | Argument 5 |
| `R9` | Argument 6 |

- `0` — `read`
- `1` — `write`
- `2` — `open`
- `9` — `mmap`
- `39` — `getpid`
- `41` — `socket`
- `60` — `exit`

`write()` syscall is `1` (`0x1`), which can output bytes to standard output.

## File Descriptors

Programs pass file descriptors to kernel system calls like `read()`, `write()`, `close()`, `dup()`, and `poll()` rather than repeatedly specifying file paths.

```text
0 - stdin
1 - stdout
2 - error
```

## printf

Integer / Pointer Argument Registers (System V x86-64) for `printf`:

| Position | printf Role | 64-bit Register |
|---|---|---|
| **1st** | `format` string pointer (`const char *`) | `rdi` |
| **2nd** | 1st format value (or 1st non-float arg) | `rsi` |
| **3rd** | 2nd format value | `rdx` |
| **4th** | 3rd format value | `rcx` |

## Position-Independent Shellcode

Now in the disassembly we find the address of shellcode. It changes with every execution, so our shellcode should find its offset relative to something and use it (position independent).

Instruction offset is calculated using RIP.

### RIP-relative Addressing

```asm
0x000000000000171f <+472>: mov %rax,0x2912(%rip)  # 0x4038
0x0000000000001726 <+479>: mov 0x290b(%rip),%rax # 0x4038
```

```text
Current instruction:  0x171f
Next instruction:     0x1726
Displacement:         0x2912
```

The CPU calculates:

```text
0x1726 + 0x2912 = 0x4038
```

`0x4038` is the location of the global variable `shellcode`.

The relative distance between the instruction and the global variable remains constant when the executable is relocated by PIE.

## Shellcode Buffer and Size

Shellcode buffer begins at `RBP-0x1010`.

Now the `shellcode_size` is here:

```asm
0x0000000000001766 <+543>: mov %rax,0x28c3(%rip)  # 0x4030 <shellcode_size>
0x000000000000176d <+550>: mov 0x28bc(%rip),%rax # 0x4030 <shellcode_size>
0x0000000000001774 <+557>: test %rax,%rax
```

```text
0x4038 = address of the global variable shellcode

Value stored at 0x4038 = address of shellcode_buffer on the stack
```
