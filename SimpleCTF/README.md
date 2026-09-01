# Simple CTF - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Simple CTF
- **Difficulty:** Easy
- **Category:** Web Exploitation & Boot2Root
- **Tools Used:** Kali Linux

## Challenge Overview
This challenge walks through hacking into a beginner-friendly Linux server with an outdated website and poor security settings.

---


## Recon

<details>
<summary>Click here</summary>
<br>
We use nmap to find which ports are open:
<br>
  
```bash
nmap -Pn -sC -sV -T4
```


- -Pn : Skip ping scanning
- -sC : Use default scripts
- -sV : Detect service versions
- -T4 : Faster scan



<img width="813" height="570" alt="nmap-scan" src="https://github.com/user-attachments/assets/039ae296-4e8a-4c26-bfee-8127674a4287" />


We found the following ports:

- 21/ftp
- 80/http
- 2222/ssh


> 1. How many services are running under port 1000? *
> 2. What is running on the higher port? s**

</details>

---



## Exploitation
<details>
<summary>Click here</summary>
 <br> 
Let's explore the open ports. We'll start with the FTP service (port 21).
<br>
  
### FTP

```bash
ftp <target ip address>
```


<img width="1341" height="551" alt="ftp-listing" src="https://github.com/user-attachments/assets/23d5806d-4c5a-437d-ad4b-e4e42d8442d3" />


I found a directory named `pub` at the FTP root, so we:

```bash
cd pub
```


Inside there is a text file named `ForMitch.txt`. Let's open the file.


<img width="1336" height="45" alt="formitch-txt" src="https://github.com/user-attachments/assets/d7e7788c-a1ce-490b-95dc-04d5311fbdd6" />


We have several clues here:

1. Username: `mitch`
2. He uses a weak password (we can crack this using a wordlist like rockyou.txt)
3. He reuses the same password elsewhere





Now, let's explore other open ports.

### HTTP
<br>

We found a `robots.txt` file on the HTTP service. Let's look at it:


<img width="905" height="620" alt="robots-txt" src="https://github.com/user-attachments/assets/92ddcec0-51b0-4e4f-b950-6f70afa89db7" />


We found several clues here:

1. There is a username `mike`
2. There is a page `/openemr-5_0_1_3`


The `/openemr-5_0_1_3` entry seems interesting. Let's inspect it:


<img width="837" height="228" alt="openemr-page" src="https://github.com/user-attachments/assets/7d4274cd-8436-47af-a4d9-6edd0ffd1f5b" />


Next, let's run gobuster to discover directories:

```bash
gobuster dir -u http://<target ip address>/ -w /usr/share/wordlists/dirb/common.txt
```


<img width="738" height="425" alt="gobuster" src="https://github.com/user-attachments/assets/3ca04d5b-fb5c-45b6-9eff-e794a3521920" />


We found the `/simple` directory. Let's see what's inside:


<img width="1916" height="882" alt="cms-made-simple-index" src="https://github.com/user-attachments/assets/5f53cb9c-c465-4f42-8f12-dffaffc55b92" />


Scrolling down a bit we found version information:


<img width="341" height="121" alt="version-info" src="https://github.com/user-attachments/assets/2e72cc16-c9ef-4dd1-b84e-610b8b50661c" />


Now that we know the software name and exact version (CMS Made Simple version 2.2.8), we can search for known vulnerabilities.

Run:

```bash
searchsploit "CMS Made Simple" 2.2.8
```

and:

```bash
searchsploit -p 46635
```

The `-p` option does two things:

1. Shows detailed info for the exploit ID (metadata, CVE, author, Exploit-DB link).
2. Prints the absolute path to the exploit file on your Kali system.


The search result points to an Unauthenticated SQL Injection (SQLi) exploit:

- Vulnerability: CMS Made Simple < 2.2.10 - SQL Injection
- CVE ID: CVE-2019-9053
- Exploit Script: 46635.py


> 3. What's the CVE you're using against the application? CVE-2019-9053
> 4. To what kind of vulnerability is the application vulnerable? SQL Injection


After identifying the CVE, copy the exploit file into your current folder:

```bash
searchsploit -m 46635
```

Because the original script was written for Python 2, convert it to Python 3 using Kali's 2to3 tool so it runs without syntax errors:

```bash
2to3 -w 46635.py
```

Now, run the exploit:

```bash
python3 46635.py -u http://<TARGET_IP>/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```


After running the script, the exploit extracts details from the database:


<img width="407" height="87" alt="extracted-data" src="https://github.com/user-attachments/assets/70bbd833-a3b6-4db2-98f0-be8c2f1e2536" />


We obtained a hashed password, but the script doesn't crack it. We can crack it with `hashcat`.

```bash
hashcat -m 20 -a 0 "0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2" /usr/share/wordlists/rockyou.txt
```

Once Hashcat finishes, show the cracked password:

```bash
hashcat -m 20 "0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2" --show
```


> 5. What's the password? s*****

</details>

---


## Login via SSH

<details>
<summary>Click here</summary>
<br>
  
After obtaining the username and password, we can SSH in on the custom port 2222:
<br>

<img width="777" height="340" alt="ssh-login" src="https://github.com/user-attachments/assets/e2166f78-6a6e-4309-a8ad-9855799f382b" />


Now let's explore the system to answer some questions:


<img width="711" height="123" alt="user-home" src="https://github.com/user-attachments/assets/8405f924-f59e-4e56-9e9e-5a9eba7ff8c4" />


> 6. What's the user flag? *******
> 7. Is there any other user in the home directory? What's its name? ******

</details>

---


## Privilege Escalation

<details>
<summary>Click here</summary>
<br>
Often we can get a root shell by abusing misconfigurations. Checking `sudo` rights is a good first step:
<br>
  
```bash
sudo -l
```


<img width="515" height="55" alt="sudo-list" src="https://github.com/user-attachments/assets/030d78f5-2452-4517-8ba5-5730fa30aee0" />


This shows that `mitch` can execute `vim` as root without a password.

Vim can spawn a shell, so running it with `sudo` allows spawning a root shell:

> 8. What can you leverage to spawn a privileged shell? vim (sudo)


Once inside vim as root, spawn a shell with:

```bash
:!bash
```


<img width="436" height="67" alt="root-shell" src="https://github.com/user-attachments/assets/faece0e8-18f7-4979-bc56-edaa4ad0c84b" />


Finally, read the root flag:

```bash
cat /root/root.txt
```


</details>
