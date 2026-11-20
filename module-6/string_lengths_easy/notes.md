# Memory Corruption

## Stack Layout

```text
RSP - 0x7ffe34a5cbf0
RBP - 0x7ffe34a5ccb0

26 8-byte words in our stack frame (208 bytes)

input buffer - 0x7ffe34a5cc10
from here its 118 bytes buffer

Saved RIP - 0x7ffe34a5ccb8
(168 bytes after the start of your input buffer)
```

## unsigned long

Like all unsigned types in C, `unsigned long` arithmetic obeys modulo arithmetic.

If a calculation exceeds `ULONG_MAX`, it silently wraps around to 0 and starts over.

Conversely, subtracting 1 from 0 will wrap around to its absolute maximum value.

## malloc()

It allocates memory on the heap and uses size (4096 here), so we can allocate 4096 bytes here.

It's stored in `[*tmp_input]`.

## read()

No Null-Terminator.

`read()` treats data purely as raw binary blocks. It will not append a `\0` string terminator to your buffer.

## strlen()

If input data lacks a null byte, `strlen()` keeps reading past the end of the buffer, which can cause a crash/security vulnerability.

Use `strnlen(3)` instead if the buffer size is fixed/untrusted.

## memcpy()

```c
void *memcpy(void *restrict dest, const void *restrict src, size_t n)
```

- Memory areas must not overlap.
- **Buffer Overflows:** `memcpy` does not check boundaries. If `n` is greater than the allocated size of `dest`, it can overwrite adjacent memory.
- **Null Terminators Ignored:** Unlike `strcpy`, `memcpy` doesn't stop at `\0`. It copies exactly `n` bytes.

## restrict

Makes sure that through the lifetime of a pointer, the object it points to will only be accessed through that specific pointer (or pointers derived directly from it).

- Instead of the compiler loading a value again and again, if it's restricted, then it just loads it once and uses it because we promise that it doesn't overlap with anything.

## Another Run

```text
RSP - 0x7ffc4c7a8360
base pointer points to 0x7ffc4c7a8420

input buffer - 0x7ffc4c7a8380

Saved RIP - 0x7ffc4c7a8428
168 bytes after the start of the input buffer.

input at least 176 bytes to overwrite it
```

## Target Instruction

```asm
0x000060fe1916a0c7 <+28>: lea 0x1022(%rip),%rdi
```

- Offset is `0c7` fixed, overwrite the last 2 bytes and keep repeating until the address matches completely.
- Or use the output, as it gives where `win_authed` is, then calculate the required offset to the target instruction.
- Received variable is also on the stack, so preserve it so you don't overwrite it.


also from disassembly 
```
received = [RBP - 0x14]
```

Input is at 
lea -0xa0(%rbp),%rax

therefore Recieved is at 140 bytes from the input 

## memcpy() Arguments

```text
RDI — destination (&data.input)
RSI — source (tmp_input)
RDX — number of bytes to copy (received)
```

So `tmp_input` would be:

```text
data ........ NULL ...... reserved received ...... data .... overwrite value
```
