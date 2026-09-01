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




