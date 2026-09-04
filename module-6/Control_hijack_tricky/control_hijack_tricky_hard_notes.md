# Control Hijack Hard

## Disassembly of challenge

```asm
0x0000000000402313 <+4>:     push   %rbp
0x0000000000402314 <+5>:     mov    %rsp,%rbp
0x0000000000402317 <+8>:     sub    $0x60,%rsp

............

0x0000000000402376 <+103>:   mov    -0x8(%rbp),%rdx
0x000000000040237a <+107>:   lea    -0x40(%rbp),%rax
0x000000000040237e <+111>:   mov    %rax,%rsi
0x0000000000402381 <+114>:   mov    $0x0,%edi
0x0000000000402386 <+119>:   call   0x401120 <read@plt>
```

Input → `-0x40(%rbp)`

RIP → `rbp + 8`

## win_authed()

Flag gets READ here:

```asm
mov    0x2dad(%rip),%eax        # 0x405040 <flag_fd.5701>
mov    $0x100,%edx
lea    0x2dc1(%rip),%rsi        # 0x405060 <flag.5700>
mov    %eax,%edi
call   0x401120 <read@plt>
```

Equivalent C:

```c
read(flag_fd, flag, 0x100);
```

`flag_fd` = file descriptor for `/flag`

`flag` = memory buffer where flag contents are stored

`0x100` = maximum 256 bytes

## Authentication Check

The check is done here:

```asm
0x0000000000402201 <+15>:    cmpl   $0x1337,-0x4(%rbp)
0x0000000000402208 <+22>:    jne    0x40230c <win_authed+282>
```

If the value is not `0x1337`, execution jumps to `0x40230c`.

The code after the check starts at:

```asm
0x000000000040220e <+28>:    lea    0xdf3(%rip),%rdi        # 0x403008
0x0000000000402215 <+35>:    call   0x4010e0 <puts@plt>
```

We jump to `0x40220e` to skip the check.

## Flag gets PRINTED here

```asm
mov    0x2e78(%rip),%eax        # flag_length
cltq
mov    %rax,%rdx
lea    0x2d6c(%rip),%rsi        # flag
mov    $0x1,%edi
call   0x4010f0 <write@plt>
```

Equivalent C:

```c
write(1, flag, flag_length);
```

## Offset

`-0x40` is 64 bytes.

```text
64 bytes
↓
saved RBP
↓
+8 bytes
↓
saved RIP
```

```text
64 + 8 = 72 bytes
```

The last 8 bytes should be `0x40220e`.
