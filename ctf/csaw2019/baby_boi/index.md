---
layout: default
title: CSAW CTF Qualification 2019 - Baby Boi
---

<h1 align="center">[CSAW CTF Qualification 2019 - Baby Boi]</h1>
## Challenge Description
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/csaw2019_baby_boi/challdesc.png"></p><br>
We are given 3 files which can be found <a href="https://github.com/ArkAngels/CTF-Source-Codes/tree/master/CSAW%20Qual%202019/pwn/CSAW%20Qual%202019%20-%20baby%20boi">here</a><br>
For some reason there is a challenge that gives us the C source code in international CTF. But that's okay and that helps the writer a lot since the writer just started to learn Binary Exploitation recently.<br>
But for now, let's try to check the file.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/csaw2019_baby_boi/checksec.png"></p><br>
It looks that the binary's configurations are as the following:<br>
* The binary is 64-bit and not stripped.
* Partial RELRO: RELRO is Relocation Read-Only. This configuration means that the *Global Offset Table* (GOT) and *Procedure Linkage Table* (PLT) is not in *read-only* mode.
* No Canary found: Canary in stack is used to detect stack smashing. When you do a stack smashing, the canary address will change because of that, and if the program detects a difference in the canary, the program will be terminated. In this case because there is no canary means we don't have to leak the canary address in order to do stack smashing.
* NX Enabled: NX itself stands for *Non-Executable*. This configuration means that every segment in the memory is not writable **AND** executable. If this configuration is enabled, we can't execute a code that comes from outsite (like if we write a shellcode).
* No PIE: PIE stands for Position Independent Executable. This configuration tells us that those addresses will change everytime we execute the ELF. But since the PIE is disabled, almost all addresses are static.
Let's try to run the program first.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/csaw2019_baby_boi/test_run.png"></p><br>
It seems the program prints an address after we input something, but what address is it?<br>
The disassembler gives us the main function like this:<br>
```C
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4; // [sp+10h] [bp-20h]@1

  setvbuf(_bss_start, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  puts("Hello!");
  printf("Here I am: %p\n", &printf, argv);
  gets(&v4);
  return 0;
}
```
