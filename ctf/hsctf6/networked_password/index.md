---
layout: default
title: HSCTF6 - Networked Password
---

<h1 align="center">[HSCTF6 - Networked Password]</h1>

## Challenge Description:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/challdesc.png"></p><br>
## Hint:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/hint.png"></p><br>

We are given a website.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/index.png"></p><br>
Hmm... Seems like it's just a simple form. Let's take a look at the page source see if we can find anything.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/page_source.png"></p><br>
Nothing, just a form which points to itself and an input box. What now? :)<br>
Well, let's just try to send something to the server and see the response.
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/fuzzing.png"></p><br>
It seems, the <i>password</i> is the only thing sent to the server and if we don't provide the correct password it will return "Invalid Password". But what is the password? How do we find it?<br>
According to the challenge description, at some point the website's response is so damn slow. So, after some fuzzings, the writer found something interesting.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/requests.png"></p><br>
When we send 2 requests, one with random string and one with not-so random string, it gives 2 very different response time.<br>
Random input:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/random_input.png"></p><br>
Not-so random input:
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/specified_input.png"></p><br>
It seems when we input a random string, which is totally wrong, it gives a shorter response time. But, when we input a <i>specified</i> string it gives us a totally different response time. Looks like this is a bruteforce with timing attack problem. And like <a href="https://arkangels.github.io/ctf/hsctf6/super_secure_system/">Super Secure System</a> problem, we have to bruteforce the flag. But knowing this is a timing attack, it will take a fucking long time. To automate the process, we can use this python script.<br>
```python
import requests, string

url = 'https://networked-password.web.chal.hsctf.com'
charset = string.letters + string.digits + string.punctuation
# print charset
flag = "hsctf{"
resp = 0
char = ""

while flag[-1] != "}":
    for i in charset:
        payload = {"password":flag + i}
        # print "[*] Trying: " + flag + i
        r = requests.post(url, data = payload)
        if r.elapsed.total_seconds() > resp:
            resp = r.elapsed.total_seconds()
            char = i
    flag += char
    resp = 0
    print "[+] Flag: " + flag
```
The script simply make a bruteforce request and capture the character which return the highest response time. In here, the writer split the bruteforce process into parts because of that shitty laptop, the bruteforce process took like more than 2 hours. But here is the last part of the bruteforce.<br>
<p align="center"><img src="https://arkangels.github.io/ctf/assets/hsctf6_networked_password/flag.png"></p><br>
Flag: hsctf{sm0l_fl4g}
