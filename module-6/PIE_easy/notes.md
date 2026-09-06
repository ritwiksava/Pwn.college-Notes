# PIE Leak + Fixed Offsets

## PIE and Memory Pages

Inside a memory page, we have functions with fixed offsets.

Memory pages are padded by `0x1000` and the base address can be randomized.

However, `main`, `challenge`, and `win` all move by the same amount because they are inside the same PIE binary.

```text
             SAME PIE LOAD BASE
                   ↓
    ┌─────────────────────────────┐
    │                             │
    │  main       + 0x1200        │
    │  challenge  + 0x1500        │
    │  win        + 0x2337        │
    │                             │
    └─────────────────────────────┘
                   ↑
             entire binary
              moves together
```

The offsets between functions are determined when the binary is compiled.

Example:

```text
Execution 1:

0x555555554000 + 0x2337
= 0x555555556337

Execution 2:

0x62a8c1700000 + 0x2337
= 0x62a8c1723337
```

The PIE base changes, but the offset remains the same.

The lower 12 bits remain the same because:

```text
0x1000 = page size
```

So:

```text
0x2337
   ↑
lower 12 bits = 0x337
```

## Finding Fixed Offsets

To see the binary's fixed addresses:

```bash
objdump -d ./binary-exploitation-control-hijack-2
```

Using GDB, we got:

```text
PIE base = 0x601d845b8000
win_authed offset = 0x1a0f
```

The instruction we need to jump to has offset:

```text
0x1a2b
```

Runtime address:

```text
runtime address = PIE base + ELF offset
```

So the target address is:

```text
0x601d845b8000 + 0x1a2b
```

## Buffer Offset

The input buffer is:

```text
buffer → saved RIP
```

The buffer starts at `RBP-0x40`.

Saved RIP is at `RBP+8`.

```text
0x40 + 0x8
= 0x48
= 72 bytes
```

So 72 bytes are needed to reach the saved RIP.

## PIE Leak

The previous approach was not the correct one.

Here we get a leak of the saved RIP.

The code does:

```asm
mov    %rbp,%rax
add    $0x8,%rax
mov    %rax,%rsp
```

`RSP` is now pointing to the saved RIP.

Then:

```asm
mov    (%rax),%rax
mov    %rax,%rsi
...
call   printf
```

The value stored at the saved RIP is loaded and passed to `printf`.

This prints the saved RIP.

This is the leak.

## Finding the PIE Base

In `main` we have:

```asm
0x2181: call   challenge
0x2186: mov    $0x0,%eax
```

The `call` instruction pushes the address of the next instruction onto the stack.

Therefore:

```text
saved RIP = PIE base + 0x2186
```

So:

```text
PIE base = saved RIP - 0x2186
```

Once we know the PIE base, we can calculate the runtime address of any other instruction using its fixed offset.

## Finding the Target Address

The target instruction is at PIE-relative offset:

```text
0x1a2b
```

Therefore:

```text
target = PIE base + 0x1a2b
```

## Understanding the Saved RIP Offset

From the static disassembly:

```asm
main:

...

0x2181: call   challenge
0x2186: mov    $0x0,%eax
```

The return address is `0x2186` because `call` pushes the address of the next instruction.

If the start of `main` is:

```text
0x2100
```

then:

```text
0x2186 - 0x2100
= 0x86
```

So the saved RIP is:

```text
main + 0x86
```

The important point is that the saved RIP contains the runtime address of the instruction at PIE-relative offset `0x2186`.

```text
Saved RIP
    ↓
PIE base + 0x2186
    ↓
leaked runtime address
    ↓
subtract 0x2186
    ↓
PIE base
    ↓
add target offset 0x1a2b
    ↓
target runtime address
```

## Exploit Calculation

```text
Leaked saved RIP
        ↓
saved RIP - 0x2186
        ↓
PIE base
        ↓
PIE base + 0x1a2b
        ↓
target address
```

The payload structure is:

```text
72 bytes
↓
saved RIP
↓
PIE base + 0x1a2b
```
