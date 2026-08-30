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


we found port
- 21/ftp
- 80/http
- 2222/ssh

1. How many services are running under port 1000? *
2. What is running on the higher port? s**

## Exploiration

Letx explore the open port. I start with exploring the ftp port first (21)

```bash
ftp <target ip address>
```



