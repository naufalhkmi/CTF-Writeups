# Operation Coldstart - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Do Not Disturb
- **Difficulty:** Medium
- **Category:** Web Exploitation & Boot2Root
- **Tools Used:** Kali Linux, BurpSuite, Nmap, feroxbuster, netcat, wordlist

## 1. Challenge Overview
The scenario places us in a test against Byte Lotus, a resort booking system running on an insecure Linux server. The web app has bad input filters and leaves internal tools open.

The main objective is to bypass the login page from the outside, get access to a standard user account to read the user flag, and then use system misconfigurations to read the final root flag.

<img width="602" height="495" alt="image" src="https://github.com/user-attachments/assets/c1ab8c9c-a60a-4362-bbe4-07e51df78f3b" />

## 2. Recon
In this step, we scan the target machine to find open ports and discover what services are running. We use Nmap to gather information about the system and check the exact software versions.

Here is what our command options do:

- Pn: Skips the initial ping check and treats the host as online.

- sC: Runs default Nmap security scripts to detect common vulnerabilities and gather extra details.

- sV: Probes open ports to determine the exact service and version numbers running on them.

- T4: Sets the scan timing to aggressive speed so the scan completes faster without being too unstable.

```bash
nmap -Pn -sC -sV -T4
```

<img width="687" height="120" alt="image" src="https://github.com/user-attachments/assets/ffbb03fd-3b22-46f1-ace1-fe79d1f04e24" />


From this scan, we discover the open services on the target:

- Port 22 (SSH): An OpenSSH service used for remote terminal administration.

- Port 80 (HTTP): A web server hosting the Node.js web application named "Byte Lotus Poolside".

Next, we look for hidden files and directories on the web server using Feroxbuster with a standard wordlist:

```bash
feroxbuster -w /usr/share/wordlists/dirb/common.txt -u http://donotdisturb.thm/
```

Hidden Endpoint Found: The tool discovers the /staff path on the web server.

Authentication Barrier: Visiting /staff redirects back to the index login page, which hints at a valid username: attendant.
---

























