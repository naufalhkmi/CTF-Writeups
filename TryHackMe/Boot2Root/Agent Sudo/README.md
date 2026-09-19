# Agent Sudo - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Agent Sudo
- **Difficulty:** Easy
- **Category:** Boot2Root
- **Tools Used:** 

## Challenge Overview

You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.
---


## Enumerate

<details>
<summary>Click here</summary>
<br>

As usuall, let us start with nmap scanning

```bash
nmap -Pn -sC -sV -T4 <ip address>
```

<img width="786" height="312" alt="image" src="https://github.com/user-attachments/assets/0be9a3ac-bd90-469a-9a75-9389f1fc51fe" />

there is 3 port open. 

> **question:** How many open ports?
> **answer :** 3


now we access the website

<img width="792" height="236" alt="image" src="https://github.com/user-attachments/assets/e2089df6-1156-4a9f-a779-33cfa711cb74" />


based on this information, we understand that, if we want to acceess the site, we must access as "user-agent".

> **question :** How you redirect yourself to a secret page?

> **answer :** user-agent

I looked at source code, didn’t find anything. I also did a go buster scan to search for other directories. Nothing

so let us use burpsuite to intercept the request.

open the burpsuite and make sure intercept is on before access the ip address machine.

<img width="1025" height="232" alt="image" src="https://github.com/user-attachments/assets/d3a3790e-e02b-4a56-a22e-64eef221842c" />


as you can see, there is user agent field. try change to letter "R" to see whether agent R is really an "agent" or not

<img width="651" height="190" alt="image" src="https://github.com/user-attachments/assets/d7fb26d9-3cab-4600-9f5d-19526912f28d" />


agent R is really an agent !. based on the information , it state "Are you one of the 25 employees?". this is number of letter. 

i try to change the user agent field into letter A. but nothing happen. same goes with letter B. but when i try letter C

<img width="856" height="143" alt="image" src="https://github.com/user-attachments/assets/857f9cac-7e27-4c06-9baf-477fdbdb6fee" />

we found another answer from this information.

> **question:** What is the agent name?

> **answer:** chris

</details>

---



## Hash cracking and brute-force

<details>
<summary>Click here</summary>
 <br> 

We know that chris use a weak password (based on previous information). and In nmap scan we found that a ftp port is open. we can use hydra on this port to find Chris's password.

```bash
hydra -l <username> -P /usr/share/wordlists/rockyou.txt -f <ip address> ftp -t 64 
```

<img width="1678" height="187" alt="image" src="https://github.com/user-attachments/assets/3238463b-4e0f-48eb-b027-db711402e0f0" />

> **question:** FTP password

> **answer:** crystal

after that, login into ftp.

we will found 3 files. donwload it.

<img width="646" height="261" alt="image" src="https://github.com/user-attachments/assets/47dde103-dda9-4491-9c24-59656bb64e6f" />


read file agentJ.txt to find more information

<img width="1510" height="121" alt="image" src="https://github.com/user-attachments/assets/47d48741-383f-4a8d-b1dd-389aa92e4592" />

so it means , another clue must be in these fake picture. 

i try steghide and exiftool nothing happen. so another option is ```binwalk```

<img width="1005" height="148" alt="image" src="https://github.com/user-attachments/assets/f9958e43-ad88-4a8d-8b0f-d4b00811a554" />

there is zip file inside cutie.png . let us extract it.

```bash
binwalk -e "cutie.png"
```


</details>

---


## Login via SSH

<details>
<summary>Click here</summary>
<br>
  

</details>

---


## Privilege Escalation

<details>
<summary>Click here</summary>


</details>


