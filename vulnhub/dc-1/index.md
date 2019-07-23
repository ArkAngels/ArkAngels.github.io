---
layout: default
title: DC-1
---

<h1 align="center">[DC-1]</h1>
Here I want to share my first experience with Vulnhub.<br>

We are given a vulnbox. As for now, we don't know anything about this box, including the IP address. To find the IP address, we can use a tool named <b>netdiscover</b>, with the following syntax:<br>
```bash
$ sudo netdiscover -i <interface> -r <IP range>
```

<p align="center"><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/netdiscover.png"></p>

It seems that our box runs in <b>172.20.10.9</b>. Now we have to find out which ports are open, in here we use a tool called <b>nmap</b>, with the following syntax:
```bash
$ nmap -A -v -p- -T4 172.20.10.9
```
<p align="center"><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/nmap.png"></p>

In here, we have 3 open ports which are:
* 22 for SSH (Secure Shell)
* 80 for HTTP (Containing a website based on Drupal 7)
* 111 for RPC

<b>Note: The Drupal version will be useful later</b>

Now, let's take a look at the website.

<p align="center"><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/homepage.png"></p>

It seems that it's just a simple Drupal 7 website. From the <b>nmap</b> report, we can see that in this website there is a text file named <i>robots.txt</i>. What is that? The information about this can be found on the following <a href="https://moz.com/learn/seo/robotstxt">link</a>. For now, let's take a look at the content of <i>robots.txt</i>.

<p align="center"><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/robotstxt.png"></p>

From the look of it, doesn't seem to contain anything useful.

Now for the Drupal part, remember the writer said that the Drupal version will be useful. Why? Because if we are going against an obsolete version of CMS (Content Management System) or something like that, there's a high chance that it contains bugs.

After a few minutes of googling, the writer found a bug that was in Drupal 7, called Drupalgeddon. 

<p align="center"><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/drupalgeddon.png"></p>

Well, according to the picture, it has the exploit in the Metasploit Framework. Perhaps we can use it to make our work easier. Let's fire up the Metasploit!

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/msfconsole.png"></p>

If you already have the exploit name, you can just use <b>search \<a name related to the exploit\></b> to find the exploit, choose, and then type <b>use \<path to exploit module\></b>.

After that, we have to set the parameters. To see what parameters we need just type <b>show options</b>.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/msf_options.png"></p>

Remember that the mandatory parameters will be tagged with <b>yes</b> at the <i>required</i> part. After we set all the parameters we need, we just have to type <b>exploit</b> to begin the process.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/meterpreter.png"></p>