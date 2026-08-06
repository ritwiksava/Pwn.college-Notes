checksec binexp.....

objdump -d binary-exploitation-first-overflow | grep -A80 "<challenge>"

________________________________________________________________________
RBP
local_18   (win_variable)
local_64   (input started here)

-offset was 71 bytes 
- It checked if local_18 ie win_variable was positve or not , so we overflowed it with 71 "1's"


Dump of assembler code for function challenge:
   0x0000000000401cc3 <+0>:     endbr64
   0x0000000000401cc7 <+4>:     push   %rbp
   0x0000000000401cc8 <+5>:     mov    %rsp,%rbp
   0x0000000000401ccb <+8>:     add    $0xffffffffffffff80,%rsp
   0x0000000000401ccf <+12>:    mov    %edi,-0x64(%rbp)
   0x0000000000401cd2 <+15>:    mov    %rsi,-0x70(%rbp)
   0x0000000000401cd6 <+19>:    mov    %rdx,-0x78(%rbp)
   0x0000000000401cda <+23>:    mov    %fs:0x28,%rax
   0x0000000000401ce3 <+32>:    mov    %rax,-0x8(%rbp)
   0x0000000000401ce7 <+36>:    xor    %eax,%eax
   0x0000000000401ce9 <+38>:    movq   $0x0,-0x50(%rbp)
   0x0000000000401cf1 <+46>:    movq   $0x0,-0x48(%rbp)
   0x0000000000401cf9 <+54>:    movq   $0x0,-0x40(%rbp)
   0x0000000000401d01 <+62>:    movq   $0x0,-0x38(%rbp)
   0x0000000000401d09 <+70>:    movq   $0x0,-0x30(%rbp)
   0x0000000000401d11 <+78>:    movq   $0x0,-0x28(%rbp)
   0x0000000000401d19 <+86>:    movq   $0x0,-0x20(%rbp)
   0x0000000000401d21 <+94>:    movq   $0x0,-0x18(%rbp)
   0x0000000000401d29 <+102>:   movq   $0x0,-0x10(%rbp)
   0x0000000000401d31 <+110>:   movq   $0x0,-0x58(%rbp)
   0x0000000000401d39 <+118>:   movq   $0x1000,-0x58(%rbp)
   0x0000000000401d41 <+126>:   mov    -0x58(%rbp),%rax
   0x0000000000401d45 <+130>:   mov    %rax,%rsi
   0x0000000000401d48 <+133>:   lea    0x3c1(%rip),%rdi        # 0x402110
   0x0000000000401d4f <+140>:   mov    $0x0,%eax
   0x0000000000401d54 <+145>:   call   0x401120 <printf@plt>
   0x0000000000401d59 <+150>:   mov    -0x58(%rbp),%rdx
--Type <RET> for more, q to quit, c to continue without paging--c
   0x0000000000401d5d <+154>:   lea    -0x50(%rbp),%rax
   0x0000000000401d61 <+158>:   mov    %rax,%rsi
   0x0000000000401d64 <+161>:   mov    $0x0,%edi
   0x0000000000401d69 <+166>:   call   0x401140 <read@plt>
   0x0000000000401d6e <+171>:   mov    %eax,-0x5c(%rbp)
   0x0000000000401d71 <+174>:   cmpl   $0x0,-0x5c(%rbp)
   0x0000000000401d75 <+178>:   jns    0x401da3 <challenge+224>
   0x0000000000401d77 <+180>:   call   0x4010e0 <__errno_location@plt>
   0x0000000000401d7c <+185>:   mov    (%rax),%eax
   0x0000000000401d7e <+187>:   mov    %eax,%edi
   0x0000000000401d80 <+189>:   call   0x401180 <strerror@plt>
   0x0000000000401d85 <+194>:   mov    %rax,%rsi
   0x0000000000401d88 <+197>:   lea    0x3a9(%rip),%rdi        # 0x402138
   0x0000000000401d8f <+204>:   mov    $0x0,%eax
   0x0000000000401d94 <+209>:   call   0x401120 <printf@plt>
   0x0000000000401d99 <+214>:   mov    $0x1,%edi
   0x0000000000401d9e <+219>:   call   0x401170 <exit@plt>
   0x0000000000401da3 <+224>:   mov    -0xc(%rbp),%eax
   0x0000000000401da6 <+227>:   test   %eax,%eax
   0x0000000000401da8 <+229>:   je     0x401db4 <challenge+241>
   0x0000000000401daa <+231>:   mov    $0x0,%eax
   0x0000000000401daf <+236>:   call   0x401bbc <win>
   0x0000000000401db4 <+241>:   lea    0x3a1(%rip),%rdi        # 0x40215c
   0x0000000000401dbb <+248>:   call   0x4010f0 <puts@plt>
   0x0000000000401dc0 <+253>:   mov    $0x0,%eax
   0x0000000000401dc5 <+258>:   mov    -0x8(%rbp),%rcx
   0x0000000000401dc9 <+262>:   xor    %fs:0x28,%rcx
   0x0000000000401dd2 <+271>:   je     0x401dd9 <challenge+278>
   0x0000000000401dd4 <+273>:   call   0x401110 <__stack_chk_fail@plt>
   0x0000000000401dd9 <+278>:   leave
   0x0000000000401dda <+279>:   ret
End of assembler dump.

-buffer starts at RBP-0x50 
-win_variable is at RBP-0x0c 
-offset was  68 bytes
-I sent 68 bytes which overwrote the win_variable. -The program checks if win_variable is non zero and if it is, it calls win().

jns = Jump if Not Sign (read() returned >= 0) 
lea -0x50(%rbp),%rax gets the address of the input buffer.

![[Screenshot 2026-08-03 155340.png]]   (will be in the screenshots)

___________________________________
#Vulnerability

lea -0x50(%rbp),%rax
mov %rax,%rsi
mov $0,%edi

call read

read is 
read(fd, buffer, count)

Arguments are 

RDI = fd
RSI = buffer
RDX = size


______________________________________
for gdb 
#To break at read (input)
b *0x401d69 run
run

now inspect rbp , offsets etc as needed.
