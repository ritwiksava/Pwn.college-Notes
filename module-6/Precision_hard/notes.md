Precision hard notes
assembler code for function challenge:

`rep stos %rax,%es:(%rdi)`

-used to **fill a block of memory with a specific value
-operation behind functions like `memset()`

**`%rax`**: Contains the 64-bit pattern to write into memory.  
**`(%rdi)`**: The target memory address. `%rdi` holds the memory pointer.

**`CLD` (Clear Direction Flag):** Sets DF = 0. The index register **increments** (moves forward through memory). This is used 99% of the time.


assembly at the input

```asm
call   0x401120 <printf@plt>
mov    -0x98(%rbp),%rdx
lea    -0x90(%rbp),%rax
mov    %rax,%rsi
mov    $0x0,%edi
call   0x401140 <read@plt>
mov    %eax,-0x9c(%rbp)
cmpl   $0x0,-0x9c(%rbp)
jns    0x401f12 <challenge+208>
0x0000000000401ee6 <+164>:   call   0x4010e0 <__errno_location@plt>
```

`-0x9c(%rbp)` is **not** a win variable; it is storing the **return value of `read()`**:

---

-now before read

```asm
mov -0x98(%rbp),%rdx
lea -0x90(%rbp),%rax
mov %rax,%rsi
mov $0x0,%edi
call read
```

here the args go like this

```text
RDI -> RSI -> RDX
```

now before read we see:

```text
RSI = RBP-0x90
```

and read is

```c
read(fd, buffer, count);
```

now becomes

```c
read(0, RBP-0x90, 0x1000);
```

---

WIN variable is here

```asm
mov -0x10(%rbp),%eax
test %eax,%eax
je 0x401f40
call win
```

-> `(RBP)-0x10`

---

```asm
mov -0xc(%rbp),%eax
test %eax,%eax
je ...
...
call exit
```

This one is checked first. If it is non zero the program exits before reaching the `win` check.

---

input buffer = `RBP - 0x90`  
win variable = `RBP - 0x10`

Offset:

```text
0x90 - 0x10
= 0x80
= 128 bytes
```

**Offset is 128 bytes**
