# Investigating Windows - TryHackMe Write-up

- **Platform:** [TryHackMe](https://tryhackme.com/)
- **Room:** Investigating Windows
- **Difficulty:** Easy
- **Category:** Digital Forensics
- **Tools Used:** 

## Challenge Overview

A windows machine has been hacked, its your job to go investigate this windows machine and find clues to what the hacker might have done.

This is a challenge that is exactly what is says on the tin, there are a few challenges around investigating a windows machine that has been previously compromised.

Connect to the machine using RDP. The credentials the machine are as follows:

Username: Administrator

Password: letmein123!

Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

---


## Basic System Information & Connection Setup

<details>
<summary>Click here</summary>
<br>

Lets start the virtual machine first by clicking the green "Start Machine" button.

ALso make sure you computer connected with TryHackMe network using OpenVPN.

Once the machine ready, search on you windows "Remote Dekstop Connection (RDP)".

And then click "Show Options" and paste the IP address that tryhackme give us.

in the username box, type username that tryhackme give us (Administrator")

It will show a new box that ask for a password. Type ```letmein123!``` and simply click "Yes"

---

After we in, to start our first inverstigation which is 

> Whats the version and year of the windows machine?

we can use Powershell window. 

Just simple click the Start button and scroll until find Windows Powershell.

After you in Powershell Window. just simply write

```bash
Get-ComputerInfo
```
to view the complete system details.

However, this command dumps hundreds of lines of text, which is messy and hard to read. To make it simple and only see the operating system name, run this filtered command:

```bash
Get-ComputerInfo -Property "Os*"
```
Look at the very first line of the output on your screen. It will tell you the exact edition and release year of the operating system.

**1. Whats the version and year of the windows machine?**

**Answer: Windows Server 2016**

</details>

---



## Investigating User Accounts
<details>
<summary>Click here</summary>
 <br> 

After we know the basic information, lets examine this user accounts on this macine.

> Attackers often create new accounts, modify existing ones, or promote standard users to administrator status so they can maintain full control. Checking account activity helps us see who has access and whether any legitimate users were impersonated.


First, we need to know every local account exist in this computer.

Type this command in  Powershell :

```bash
Get-LocalUser
```

Powershell will display every username, wheter the account is active and a description.

[show list of username]

Next, we want to determine which of these users signed in most recently

Type 

```bash
net user {username} |findstr “Last”
```
> The findstr "Last" part tells the system to only print the line that mentions the last logon time.

**2. Which user logged in last?**

**Answer :Administrator**


--- 

Now, look at the output you received when checking the user John. 

This shows the exact timestamp of John's final session.

**3. When did John log onto the system last?**

**Answer: 03/02/2019 5:48:32 PM**


--- 

After checking login history, we need to inspect privilege levels.

To see every account that holds full administrative permissions, run this PowerShell command:

```bash
Get-LocalGroupMember -Group "Administrators"
```
And then you will spot two accounts that normally should not have elevated privileges

**5. What two accounts had administrative privileges (other than the Administrator user)?**

**Jenny, Guest**



</details>

---


## Uncovering Persistence

<details>
<summary>Click here</summary>
<br>
  
After checking user accounts, we need to find out how the attacker stays inside the system. 

in cybersecurity, this is called persistence. When an attacker hacks a machine, they do not want to lose access if the computer restarts or if someone logs out.

To survive a reboot, they hook into startup mechanisms—the places where the operating system automatically loads programs.


One of the most popular hiding spots for persistence is the **Windows Registry**
> The Registry is a huge database where Windows stores system configuration settings. Specifically, the "Run" key tells the operating system to automatically start certain programs every time Windows boots up.

To inspect this area, open the Windows search bar, type ```regedit``` and follow this file path :

```bash
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Once you reach the Run folder, you will spot an entry named "UpdateSvc" that immediately looks suspicious

It reveals the external IP address the machine attempts to reach out to immediately when it boots up.

**4. What IP does the system connect to when it first starts?**

**10.34.2.3**

---

Another common method attackers use for persistence is the **Windows Task Scheduler**

> This tool allows programs to run automatically at set times, similar to setting an alarm clock on your phone

Let's head back to PowerShell to list all scheduled tasks.

Type command :

```bash
Get-ScheduledTask
```

Well we get a lot of tasks scheduled to run. Lets make it smaller.

```bash
Get-Scheduled | where {$_.TaskPath -eq “\”}
```

> This will only output anything with a \ TaskPath

Looking closely at the names, one task stands out: "Clean file system".

While the name sounds like a harmless system cleanup tool, the root path location makes it highly suspicious.

**6. Whats the name of the scheduled task that is malicous.**

**Clean file system**

---

Now that we know the name of the malicious task,

we need to inspect what it actually does when triggered.

We can save this specific task into a temporary PowerShell variable named ```$task``` and inspect its actions directly


```bash
$task = Get-ScheduledTask | Where TaskName -EQ "Clean file system"
```

and then

```bash
$task = Get-ScheduledTask | Where TaskName -EQ "Clean file system"
```

and we examine the output.

You will see that it is designed to run a script file named ```nc.ps1```, which is a PowerShell version of Netcat.

> a popular networking tool often used by attackers to create unauthorized connections.

**7. What file was the task trying to run daily?**

**Answer: nc.ps1**


And then, look at the Arguments column

Next to the script name, you will see extra instructions passed to the program, including a specific port number


This tells us which door the script opens on the computer to wait for the attacker's incoming commands.

**7. What port did this file listen locally for?**

**Answer: 1348**

---



</details>

---


## Tracing Attack Evidence

<details>
<summary>Click here</summary>

Now that we know the user accounts and persistence tricks, our final step is to piece together the entire attack timeline.

When a hacker breaks into a computer, they always leave traces behind, such as modified files, event logs, downloaded malware, and altered network settings.


First, we need to pinpoint the date the compromise took place.

When an attacker drops files or creates working folders on a system, the file system records the creation date.


Open File Explorer,


and click on Local Disk (C:).

and look through the list of folders and their "Date modified"

You will see an unusual temporary folder along with several folders sharing the exact same creation date that does not match normal Windows installation history.

**8. At what date did the compromise take place?**

**Answer: 03/02/2019**

---

Next we need to track down the exact second the attacker gained elevated permissions.

Windows records every security-related change inside the **Event Viewer.**

Click the Start button, type "Event Viewer", and open it

Because the system log has thousands of entries, we must filter it to avoid reading everything manually.

- On the right-hand panel, click Create Custom View.
- Set the Logged drop-down to "Custom Range"
- choose the date March 2, 2019, from 4:00:00 PM to 4:30:00 PM
  > because we know from earlier clues that activity happened around late afternoon
- Under the Event Logs section, expand Windows Logs and check Security, then click OK
- A Window will pop-up to Save the Filter and ask you to name it, just click the OK button on the right side of the Window.

<br>

- We have 297 Events to look through, let us start at the beginning.
- Once at the bottom, you can slowly start to scroll up. as you do look at the Task Category.
- This will indicate what we are looking for, which is privilege changes to a new logon
- You are looking for **Security Group Management**, once you find that, look to the Date and Time

**9. At what time did Windows first assign special privileges to a new logon?**

**Answer: 03/02/2019 4:04:47 PM**

---

Next, we look for the tools the attacker used to steal credentials.

During our investigation, a command window briefly showed an application path inside ```C:\TMP\mim.exe```

In File Explorer, navigate to Local Disk (C:) > TMP

Inside this folder, you will find several suspicious files, including 
- mim.exe and
- mim-out

Double-click mim-out to open it in Notepad.

The header text inside clearly reveals the name of a very famous open-source tool used by attackers to pull plaintext Windows passwords out of system memory.

**10. What tool was used to get Windows passwords?**

**Answer: mimikatz**

---

Now, let's track down the attacker's Command and Control (C2) server

which is the computer the attacker uses to send orders to their malware

Computers use a small file called the ```hosts``` file as a private address book to match website names to IP addresses.

Hackers often edit this file to trick your computer into sending traffic to their own fake server instead of the real website. - a technique known as **DNS poisoning**

Navigate in File Explorer to:

```bash
C:\Windows\System32\drivers\etc
```

Open host file with notepad.

look for a strange ip address. you will spot a famous search engine domain paired directly with an unfamiliar IP address.

normal computers never hardcode popular search engines into this file.

That weird IP address belongs to the attacker's server, and the domain name beside it is the website the hacker hijacked.

**11. What was the attackers external control and command servers IP?**

**Answer: 76.32.97.132**


--- 

we also need to know the very first way the attacker managed to break into the computer.

When a Windows computer hosts a website, it usually keeps all its public web pages inside a specific folder called ```inetpub```.

Open File Explorer and go here:

```bash
Local Disk (C:) → inetpub → wwwroot
```

Inside this wwwroot folder, you will spot three files, two of which is .JSP files and the other is a .GIF file.

the .GIF file isn’t super suspicious on a computer, but what is a .JSP file? With a quick google search we find out.

> A .jsp file stands for JavaServer Page. It is a text-based document used by developers to create dynamic, interactive web pages based on the Java programming language.


So what is a Java file doing here, this seems like it could be the file extention we are looking for.

> - What are these script files? Attackers secretly upload them through weak website forms (like a broken file upload feature). These malicious scripts are called web shells.

> - What do they do? They work like a backdoor, letting the hacker type and run system commands from their own browser.

> - What should you check? Look at the very end of the filenames (the file extension, such as .aspx or .php). This shows you the programming language the hacker used to write their backdoor.

**12. What was the extension name of the shell uploaded via the servers website?**

**Answer: .jsp**


--- 

To ensure they could keep connecting directly to the server from outside,

the attacker also punched a hole through the computer's firewall.

Let's see what network port they opened.

Click the Start button, type "firewall", and open **Windows Defender Firewall** with Advanced Security.

In the left panel, click Inbound Rules (traffic coming into the PC).

To hide the hundreds of standard Windows rules, look at the right-hand panel,

- click Filter by Group,
- and select Rules without a Group.

You will see a custom rule labeled "Allow outside connections for development".

Double-click it, go to the ```Protocols and Ports tab```,

and read the local port number listed inside.

**What was the last port the attacker opened?**

**Answer: 1337**


---

**DNS Poisoning**

Normally, when you type a website name like google.com into a browser, your computer asks an internet directory (called a DNS server) for the real IP address.


However, before the computer asks the internet, it always checks its own local address book first—the hosts file located at:

```bash
C:\Windows\System32\drivers\etc\hosts
```

If someone writes a website name inside this file and points it to a different IP address, your computer will blindly trust it and go to the wrong destination.

this trick is called **DNS poisoning** or **hosts file hijacking**


To check which site was targeted, open File Explorer and follow the path to

```bash
C:\Windows\System32\drivers\etc
```

> A clean Windows hosts file usually contains only comments (lines starting with #) or default local entries like 127.0.0.1 localhost.


You will see fake entry, popular, well-known search engine domain is manually mapped to the suspicious IP address 76.32.97.132 (the attacker's C2 server).

**Check for DNS poisoning, what site was targeted?**

**Answer: google.com**
</details>










