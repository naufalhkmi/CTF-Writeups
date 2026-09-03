<img width="1067" height="638" alt="image" src="https://github.com/user-attachments/assets/58ea2650-fd0c-4144-87cc-44620e4a12dc" /># Crack The Hash - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Crack The Hash
- **Difficulty:** Easy
- **Category:** 
- **Tools Used:** 

## Challenge Overview

Cracking hashes challenges

---


**METHOD CRACK HASH PASSWORD**

## Online Decryption

-  [100L5](https://10015.io/tools/)
-  [MD4](https://www.dcode.fr/md4-hash)

## Using hashcat

- cybersecurity tool used to recover or "crack" forgotten passwords from their scrambled forms (called hashes).
- It supports over 300 different hashing algorithms
- [hash mode](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [hash mode -github](https://github.com/unstable-deadlock/brashendeavours.gitbook.io/blob/master/pentesting-cheatsheets/hashcat-hash-modes.md)
## Identifying Hashing Algorithms

- [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/)
- hashid or hash-identifier tools

---

# LEVEL 1
<details>
<summary>Click here</summary>
<br>

1. 48bb6e862e54f2a795ffc4e541caed4d 

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool

<img width="997" height="746" alt="image" src="https://github.com/user-attachments/assets/943000b2-5654-4408-9f69-1cf75c74016b" />


Its MD5 type. now let use [100L5](https://10015.io/tools/) to decrypt 

<img width="1285" height="216" alt="image" src="https://github.com/user-attachments/assets/db1b2167-2dae-41b9-b614-08015a48bc9a" />

<br>

---

2. CBFDAC6008F9CAB4083784CBD1874F76618D2A97

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool


Its SHA1 type. now let use [100L5](https://10015.io/tools/) to decrypt 

<img width="1282" height="202" alt="image" src="https://github.com/user-attachments/assets/f9a48ff4-e190-4650-9d61-c0b2eb1497e4" />


<br>

---

3. 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool



- Its SHA-256 type. now let use [100L5](https://10015.io/tools/) to decrypt 

<img width="1287" height="267" alt="image" src="https://github.com/user-attachments/assets/05d26630-cfc4-4138-96b6-4df2b14c8bde" />


<br>

---

4. $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool

  <img width="935" height="625" alt="image" src="https://github.com/user-attachments/assets/d2f89f99-a1d6-41db-925b-6a2862eea2ba" />

- its bcrypt type. 
- online lookup sites usually do not have precomputed Bcrypt tables

So let decrypt this hash in our machine

- i start filter the wordlist for 4-character words

```bash

grep -E '^.{4}$' /usr/share/wordlists/rockyou.txt > 4char.txt
```
<br>

and then i insert the hash code into file hash.txt to make it look clean when i type next command

<br>

```bash
echo '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom' > hash.txt
```

<br>

and then run hashcat command

```bash
hashcat -m 3200 -a 0 hash.txt 4char.txt
```

- -m 3200 : hash mode for bcrypt
- -a : attack mode

<br>

and then view the cracked password : 

<img width="637" height="67" alt="image" src="https://github.com/user-attachments/assets/df802b75-fbad-4bd2-b278-8d73657e6411" />

---

<br>

5. 279412f945939ba78ce0758d3fd83daa

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool

- Its MD4 type. now let use [MD4](https://www.dcode.fr/md4-hash) to decrypt

<img width="1006" height="205" alt="image" src="https://github.com/user-attachments/assets/6adfa047-c0c6-44a1-af13-e01564365bf6" />


</details>

---

# LEVEL 2
<details>
<summary>Click here</summary>
<br>

1. F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool

- Its	SHA2-256 type. now let use [100L5](https://10015.io/tools/) to decrypt

<br>

<img width="1222" height="232" alt="image" src="https://github.com/user-attachments/assets/e4541eba-e78f-4342-892f-219af780a9a6" />

<br>

Since the hash is too long. we need to use our terminal and start use hashcat command to solve this problem

If using hashcat. we need to identify the hash first so that we can decide which hashcat mode that we can use.

[hash mode -github](https://github.com/unstable-deadlock/brashendeavours.gitbook.io/blob/master/pentesting-cheatsheets/hashcat-hash-modes.md)

I use command 

```bash
hashcat -m 1400 -a 0 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85' /usr/share/wordlists/rockyou.txt
```

<img width="742" height="100" alt="image" src="https://github.com/user-attachments/assets/f3e6787f-6841-443f-a92b-336fc150c6fc" />

We found it.

---

2. 1DFECA0C002AE40B8619ECF94819CC1B

- we identify the hash by using [Tunnelsup](https://www.tunnelsup.com/hash-analyzer/) tool
 
- its MD5 / MD4 type. let use hashcat for this hash


```bash
hashcat -m 0 -a 0 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
```

This is for MD5 mode

```bash
hashcat -m 900 -a 0 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
```

This is for MD4 mode.


But when i try these 2 hashmode, both status said is "exhausted". So i try identify again this hash password using other platform which is [hashes](https://hashes.com/en/tools/hash_identifier)

It sais it is NTLM type. So hash cat mode for NTLM is ```1000```

```bash
hashcat -m 1000 -a 0 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
```

<img width="502" height="97" alt="image" src="https://github.com/user-attachments/assets/f9a37c8e-b402-466e-8540-fc532a5341db" />

---


3.  $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

Salt: aReallyHardSalt


<br>

We identify the hash type using [hashes](https://hashes.com/en/tools/hash_identifier)

We found it is sha512crypt $6$, SHA512 (Unix) type

Next, find the hash mode at [hash mode -github](https://github.com/unstable-deadlock/brashendeavours.gitbook.io/blob/master/pentesting-cheatsheets/hashcat-hash-modes.md)

```bash
hashcat -m 1800 -a 0 '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.' /usr/share/wordlists/rockyou.txt 
```



---


4. e5d8870e5bdd26602cab8dbe07a942c8669e56d6

Salt :  tryhackme

We identify the hash type using [hashes](https://hashes.com/en/tools/hash_identifier)












