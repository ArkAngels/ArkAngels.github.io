---
layout: default
title: HSCTF6 - Super Secure System
---

<h1 align="center">[HSCTF6 - Super Secure System]</h1>

## Challenge Description:
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/challdesc.png"></p><br>

We are given a network service:<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/nc.png"></p><br>

The service itself is asking for input. For example if we input as on the following picture:
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/try.png"></p><br>

It will encrypt our input with a random unknown key. And if we are being idle for a certain time, it will become time out.<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/timeout.png"></p><br>

But notice that the input itself, when we input a character the encrypted string will become 2 characters. And the encrypted message is 106 characters long. So we can conclue that the flag is 106/2 which is 53 characters. At first the writer tried to bruteforce per character to meet the requirement, but it seems that the writer was wrong. Refer to the following image:<br>
<p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/fuzzing.png"></p><br>

When we input "hs", and then we input "s", the encrypted string of "s" is different from when we input "hs". From this point, after a discussion with my teammates, we come into a conclusion that we have to bruteforce the flag. Since it has time out it will be difficult to bruteforce manually. So, I make a python script to bruteforce the flag. Since the writer don't have any experience in using <i>pwntools</i> AT ALL, the original solution script was so damn shit. But thanks to the writer's teammate <b>cipung</b> he helped simplified the code. Here is the solution scripts.<br>

```python
# Original Solution
from pwn import *
import string, sys

conn = remote("crypto.hsctf.com", 8111)
n = 2
flag = "hsctf{"

print "[+] " + conn.recvline()
print "[+] " + conn.recvline()
print "[+] " + conn.recvline()
strings = conn.recvline()
strings = strings.split(": ")[1]
print "[+] Encrypted flag: " + strings
strings = strings.split("\n")[0]
helper = strings
# print helper
strings = [strings[i:i+n] for i in range(0, len(strings), n)]
# print strings
index = 6
index_hash = 14
strings = strings[index:]
# print strings
for check_str in strings:
    for i in string.printable:
        check = conn.recvuntil("Enter the message you want to encrypt: ")
        # print check
        check = conn.sendline(flag + i)
        # print final + i
        check = conn.recvline()
        check = conn.recvline()
        # print check
        check = check.split("\n")[0].split(": ")[1]
        # print "Helper: " + helper[:index_hash]
        # print "Check: " + check
        if check == helper[:index_hash]:
            flag += i
            if len(flag) == 52:
                flag += "}"
                print "[+] Flag: " + flag
                sys.exit(1)
            print "[+] Flag: " + flag
            index += 2
            index_hash += 2
            break
```
Yeah lots of commented prints, it is because the writer need to know what string is being printed and such. The thing is, we bruteforce directly the flag. For example, the flag format is "hsctf{\<flag string\>}", so the bruteforce will start from "hsctf{" until we reach "}" that indicates the end of flag. And we know that the flag length is 53 characters. <br>

Here is the solution simplified by the writer's teammate:<br>
```python
from pwn import *
import string
pool = string.printable
r = remote('crypto.hsctf.com',8111)
r.recvuntil('Here is my super secret message: ')
cipher = r.recvline()[:-1]
print "Cipher :" + cipher
flag = 'hsctf{'
n = 2
while flag[-1]!='}':
	for i in pool:
		payload = flag+i
		r.sendline(payload)
		r.recvuntil('Encrypted: ')
		trial = r.recvline()[:-1]
		check = cipher[:(len(payload)*2)]
		if trial == check:
			flag+=i
			print "flag: {}".format(flag)
			break
```
And after some time, we get the flag.<br>
<!-- <p align="center"><img src="https://blog.xarkangels.com/ctf/assets/hsctf6_super_secure_system/flag.png"></p><br> -->
Flag: hsctf{h0w_d3d_y3u_de3cryP4_th3_s1p3R_s3cuR3_m355a9e?}
