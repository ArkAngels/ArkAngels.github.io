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
* Partial RELRO: RELRO is Relocation Read-Only. This configuration means that the *Global Offset Table* (GOT) and *Procedure Linkage Table* (PLT) is not in *read-only* mode