## Partial RIP Overwrite

```asm
0x00000000000016c4 <+15>:    cmpl   $0x1337,-0x4(%rbp)
0x00000000000016cb <+22>:    jne    0x17cf <win_authed+282>
0x00000000000016d1 <+28>:    lea    0x930(%rip),%rdi        # 0x2008
0x00000000000016d8 <+35>:    call   0x10f0 <puts@plt>
```

Target instruction offset:

```text
0x16d1
```

## PIE Base

```gdb
(gdb) info proc mappings
```

```text
process 193

Mapped address spaces:

Start Addr         End Addr           Size    Offset   Perms  File
0x000064bc68e30000 0x000064bc68e31000 0x1000  0x0      r--p   /challenge/binary-exploitation-pie-overflow
```

From this, we know the PIE base is:
0x000064bc68e30000


Runtime addresses are randomized, but the last 12 bits remain the same because the binary is page aligned.

## Partial Overwrite

We need to overwrite the lowest 2 bytes of the saved RIP.

The fourth nibble must be guessed because we can overwrite approximately 1.5 bytes.

We keep trying different values until we reach the correct address.

Example crash:

```text
Program received signal SIGSEGV, Segmentation fault.

0x00005c8d3bbf56d1 in ?? ()
```

The last three nibbles are:

```text
6d1
```

This matches the target instruction offset:

```text
0x16d1
```

So we know that the partial overwrite is reaching the correct lower bits.
The next step is to write a script that repeatedly tries the possible values until the correct address is reached.
