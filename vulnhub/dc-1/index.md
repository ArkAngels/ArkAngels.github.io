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

Yep when the exploit executed successfully, it will open a meterpreter session. In here, we haven't get any shell yet. But maybe we can do something with this meterpreter? Let's see what options do we have.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/meterpreter2.png"></p>

It seems we have the <b>shell</b> command that can be used to spawn shell.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/shell.png"></p>

Okay, now we have shell but only as <b>www-data</b>. It's okay, we will do something about it later. For now, let's explore the server for a bit.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/first_flag.png"></p>

Oh it seems we just got our first flag from this directory.

```
First flag:
Every good CMS needs a config file - and so do you
```

Okay, judging from the flag, we now have another clue. This clue refers to a config file owned by Drupal 7. The config file location for Drupal 7 can be found in <i>sites/default/</i>

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/getting_second_flag.png"></p>

There are <b>settings.php</b> and <b>default.settings.php</b>. Since we need don't need the default one because Drupal will <b>ALWAYS</b> look for <b>settings.php</b>, we can just open the file and see what's inside.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/second_flag.png"></p>

There goes our second flag.

```
Second flag:
Brute force and dictionary attacks aren't the only ways to gain access (and you WILL need access). What can you do with these credentials?
```

Okay from the second flag we got, it seems we have to look into the database. And luckily we can get the MySQL login below the second flag.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql_password.png"></p>

```
Database -> drupaldb
Username -> dbuser
Password -> R0ck3t
```

Now we can login to MySQL for some more exploring.

```
$ mysql -u dbuser -p R0ck3t
```

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql.png"></p>

After we logged in, we can find the list of tables inside <i>drupaldb</i> database.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql_tables.png"></p>
<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql_tables2.png"></p>
<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql_tables3.png"></p>
<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/mysql_tables4.png"></p>

Okay, there are lots of tables there. But knowing from the last clue, we can have a look at the <b>users</b> table.

```
mysql> SELECT * FROM users;
```

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/dump_user_table.png"></p>

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/admin_hash.png"></p>

Yep, we got the hash for admin password. From here, there are two ways we can get the next flag:
* Use hashcat + rockyou.txt to crack the password.
* Manually explore the MySQL for flag.

Since the writer's laptop is shit, we decided to use the second method. It took around almost 30 minutes to explore the tables from the top. And finally we got the third flag from <b>search_dataset</b> table.

<p align="center" ><img src="https://blog.xarkangels.com/vulnhub/assets/dc-1/third_flag.png"></p>

```
Third flag:
Special perms will help find the passwd but you ll need to exec that command to work out how to get what’s in the shadow
```