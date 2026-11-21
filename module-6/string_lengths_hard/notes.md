# Memory Corruption — Assembly and Function Arguments

## read() — Assembly

```asm
mov    -0x8(%rbp),%rdx
mov    -0x10(%rbp),%rax
mov    %rax,%rsi
mov    $0x0,%edi
call   0x1190 <read@plt>
```

On 64-bit Linux, `read` has **syscall number `0`** and uses the `syscall` instruction.

| Purpose / Argument | Register (x86-64) | Value for read |
|---|---|---|
| Syscall number | `rax` | `0` (`SYS_read`) |
| Arg 1 (`fd`) | `rdi` | File descriptor (e.g. `0` for stdin) |
| Arg 2 (`buf`) | `rsi` | Address of destination buffer |
| Arg 3 (`count`) | `rdx` | Number of bytes to read |
| Return value | `rax` | Bytes read, `0` on EOF, or `-errno` on error |

**Note:** The assembly above calls `read@plt`, which is the libc function. It passes arguments using the x86-64 function calling convention. A direct syscall uses `rax` for the syscall number and the `syscall` instruction.

## strlen() — Function Arguments

| Purpose / Argument | Register (x86-64) | Notes for strlen |
|---|---|---|
| Arg 1 (`s`) | `rdi` | Pointer to null-terminated string |
| Return value | `rax` | String length (`size_t`) |

## malloc() — Function Arguments

| Purpose / Argument | Register (x86-64) | Notes for malloc |
|---|---|---|
| Arg 1 (`size`) | `rdi` | Number of bytes to allocate (`size_t`) |
| Return value | `rax` | Pointer to allocated memory (`0` / `NULL` on failure) |

## memcpy() — Function Arguments

| Purpose / Argument | Register (x86-64) | Notes for memcpy |
|---|---|---|
| Arg 1 (`dest`) | `rdi` | Destination address |
| Arg 2 (`src`) | `rsi` | Source address |
| Arg 3 (`n`) | `rdx` | Number of bytes to copy (`size_t`) |
| Return value | `rax` | Original `dest` pointer (`rdi`) |

## Understanding the Assembly

From the assembly, we get to know:

```text
heap  - rbp-0x10  (memcpy copies from here)
count - rbp-0x8
```

## Offsets From Input Buffer

Input buffer: `rbp-0x40`

```text
strlen    - should stop after 24 bytes
received  - 44 bytes
saved RIP - 72 bytes
```

So input would be:

```text
.......\x00.......74........0x18b7
```
